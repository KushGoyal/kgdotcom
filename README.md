~~~bash
$ bundle config set path vendor/bundle
$ bundle
~~~

Run `jekyll` commands through Bundler to ensure you're using the right versions:

~~~bash
$ bundle exec jekyll serve --watch --drafts
~~~

~~~bash
$ JEKYLL_ENV=production bundle exec jekyll build
$ firebase deploy
~~~

