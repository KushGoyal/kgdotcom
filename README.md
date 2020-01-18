~~~bash
$ bundle install --path vendor/bundle
~~~

Run `jekyll` commands through Bundler to ensure you're using the right versions:

~~~bash
$ bundle exec jekyll serve --watch --drafts
$ JEKYLL_ENV=production bundle exec jekyll build
~~~

~~~bash
$ JEKYLL_ENV=production bundle exec jekyll build
$ firebase deploy
~~~

