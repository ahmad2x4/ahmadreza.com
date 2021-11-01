# Ahmadreza's Blog
Follow [this](instruction) to install jekyll on windows 


## Run it locally

The following command builds the site and runs it on http://localhost:4000/
It takes a while because I have a lot of posts.

```shell
bundle exec jekyll build
```
```shell
bundle exec jekyll serve
```

## Testing

[HTML::Proofer](https://github.com/gjtorikian/html-proofer) is set up to validate all links within the project.  You can run this locally to ensure that your changes are valid:

```shell
bundle install
bundle exec rake test
```


## issues and resolutions
+ [How to solve “certificate verify failed” on Windows?](http://stackoverflow.com/a/16134586)
+ [Gist should be installed](https://github.com/poole/hyde/issues/114)
+ [install redcarpet](https://richonrails.com/articles/rendering-markdown-with-redcarpet)