---
layout: post
date: 2019-02-21
title: Login api using Django Rest Framework Token Authentication with Google Recaptcha and token expiry
categories:
  - django
  - coding
comments: true
---

Django Rest Framework (DRF) is an amazing framework to create a REST API.
It has token based auth built in to create a login api. I have used the code from DRF to create a more
feature rich login API.

I have added:
* Token caching till the user logs out
* Time based token expiry
* Logout api
* Google recaptcha support
* password change token expiry
* Saving the last login time

The code runs on Django 2.1.x and DRF 3.8.x and 3.9.x

Lets see the changes one feature at a time.

I cached the tokens for logged in users.
This saves the database query to get the user and token in every request. The reduction in
request time is significant.
You need to create a login request to save the token in the cache and send the token 
and user details to the client. The token is used as the key and the pickled user object is used 
as the value.

```python
@api_view(['POST'])
@permission_classes([])
@throttle_classes([AnonRateThrottle])
def login(request):
    serializer = AuthTokenSerializer(data=request.data)
    serializer.is_valid(raise_exception=True)
    user = serializer.validated_data['user']
    # save the last login time
    user.last_login = timezone.now()
    user.save(update_fields=['last_login'])
    token, created = Token.objects.get_or_create(user=user)
    # save the token and user in the cache
    # set the timeout as token expiry in seconds to remove the value from cache
    cache.set(token.key, pickle.dumps(user), 
              int(os.getenv('TOKEN_EXPIRY_IN_SECONDS')))
    user_serializer = UserSerializer(user)
    return Response({'token': token.key, 'user': user_serializer.data})
```

I created `CachedTokenAuthentication` class to use cached tokens for logged in users.
To use this class you have to update the `DEFAULT_AUTHENTICATION_CLASSES` 
setting in your `REST_FRAMEWORK` settings dict.

```python
class CachedTokenAuthentication(TokenAuthentication):
    """
    checks the cache to get the token and the user.
    Then falls back to the database.
    """

    def authenticate_credentials(self, key):
        model = self.get_model()
        # get token and user from cache
        user_pickle = cache.get(key)
        if user_pickle:
            user = pickle.loads(user_pickle)
            return user, model(key, user)
        else:
            user, token = super().authenticate_credentials(key)
            # check if the token has expired
            token_age = (timzone.now()- token.created).seconds
            if token_age > int(os.getenv('TOKEN_EXPIRY_IN_SECONDS')):
                # delete the expired token
                token.delete()
                raise exceptions.AuthenticationFailed(_('Token has expired'))
        return user, token
```

To make the login api more secure I added Google Recaptcha. To do this I 
created a `CaptchaAuthTokenSerializer` class which validates the captcha response 
from the google api. In the login request post data you have to send the username, password and 
g_recaptcha_response.

```python
class CaptchaAuthTokenSerializer(AuthTokenSerializer):
    g_recaptcha_response = serializers.CharField()

    def is_recaptcha_valid(self, response):
        # https://developers.google.com/recaptcha/docs/verify
        url = 'https://www.google.com/recaptcha/api/siteverify'
        recaptcha_key = os.getenv('RECAPTCHA_KEY')
        r = requests.post(url, data={'secret': recaptcha_key, 'response': response})
        success = BooleanField().to_python(r.json().get('success'))
        return success

    def validate(self, attrs):
        attrs = super().validate(attrs)
        if not self.is_recaptcha_valid(attrs['g_recaptcha_response']):
            raise serializers.ValidationError('invalid captcha response')
        return attrs

    def update(self, instance, validated_data):
        super().update(instance, validated_data)

    def create(self, validated_data):
        super().create(validated_data)
```

Now in the above login method instead of the `AuthTokenSerializer` you have to use the 
`CaptchaAuthTokenSerializer`. Rest of the code remains same.

Next I created a logout request which deletes the token from the db and clears the cache.

```python
def delete_token(user):
    try:
        token = Token.objects.get(user=user)
        cache.delete(token.key)
        token.delete()
    except Token.DoesNotExist:
        pass
    return


@api_view(['POST'])
@permission_classes([IsAuthenticated])
def logout(request):
    delete_token(request.user)
    return Response()
```

In the password reset request I am deleting the token and creating a new token.
This is a security feature which will logout all other active sessions on a password change.

```python
@api_view(['POST'])
def change_password(request):
    form = PasswordChangeForm(request.user, request.data)
    if form.is_valid():
        form.save()
        delete_token(request.user)
        token = Token.objects.create(user=request.user)
        serializer = UserSerializer(request.user)
        return Response({'token': token.key, 'user': serializer.data})
    return Response(form.errors, status=status.HTTP_400_BAD_REQUEST)
```