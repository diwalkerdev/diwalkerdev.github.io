---
layout: post
title:  "Welcome to GitHub pages and Jekyll"
date:   2019-05-31 10:04:02 +0100
categories: jekyll github github-pages update
---
# Intro 
Did you know that you can host a website directly from your github account?

Simply create a repository called `<your-username>.github.io` and commit html files to `master` or 
`gh-pages` branches.

You can then access your site from: `https://<username>/github.io` :sunglasses:

See [github-pages][github-pages] for more information.

# Jekyll
Jekyll is a static website generator. Basically it'll turn markdown into html which is useful for
creating content quickly and easily. But you can also combine this with custom html and css to actually make your site look half-decent.


The benefit of using Jekyll over other generators (of which there are many) is that Github will 
automatically rebuild your content whenever you push changes. If you were to use a different 
generator you'd have to build the content manually and then link the generated content to your 
`<your-username.github.io>` repo. This means you end up with two repos; your github.io repo which 
purely hosts the generated content and a second which contains the static content and whatever else 
your generator needs.

See [jekyll-docs][jekyll-docs] for more information.

# Instructions
The steps to get GitHub pages and Jekyll working together are:
1. Create the repo on GitHub (see Intro).
2. Install Ruby. [jekyll-ruby-install][jekyll-ruby-install]
3. Create a Jekyll site. [jekyll-docs][jekyll-docs]
4. Run `bundle update`.
5. Push changes.
6. View your site online!

If you have issues viewing your site, either 404 errors or the content not updating correctly try the 
following:
1. Update Gemfile to include github-pages (See GitHub Pages Gem below).
2. Update the _front matter_ for index.md (see Front Matter below).
3. Your browser may be caching old pages; open your site using private browsing or clear 
the browser's cache.
4. It takes a minute for GitHub to rebuild your site - be patient!

## GitHub Pages Gem
[jekyll-github-pages][jekyll-github-pages] suggests adding to the `Gemfile`:
```
gem "github-pages", group: :jekyll_plugins
```
And to run `bundle update` often.

## Front Matter
_What's front matter?_

From [github-config-jekyll][github-config-jekyll]:
> Jekyll requires that Markdown files have front matter defined at the top of every file. Front 
matter is just a set of metadata, delineated by three dashes:

Be sure to have at least `title` and `layout` so the front matter (everything inside 
the dashes) looks something like:

```
title: Home Page
layout: home
```

And you'll be good to go :dash:

[github-pages]: https://pages.github.com
[jekyll-docs]:  https://jekyllrb.com/docs
[jekyll-gh]:    https://github.com/jekyll/jekyll
[jekyll-talk]:  https://talk.jekyllrb.com/
[github-config-jekyll]: https://help.github.com/en/articles/configuring-jekyll
[jekyll-ruby-install]: https://jekyllrb.com/docs/installation/
[jekyll-github-pages]: https://jekyllrb.com/docs/github-pages/
