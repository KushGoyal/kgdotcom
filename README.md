Run the below command to install all dependencies
~~~bash
$ bundle
~~~

Run `jekyll` commands through Bundler to ensure you're using the right versions:
~~~bash
$ bundle exec jekyll serve --watch --drafts
~~~

~~~bash
$ JEKYLL_ENV=production bundle exec jekyll build; firebase deploy
~~~

[bundler best practices](https://www.viget.com/articles/bundler-best-practices/)
