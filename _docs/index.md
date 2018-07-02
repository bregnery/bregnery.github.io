---
title: Welcome
permalink: /docs/home/
redirect_from: /docs/index.html
---

## About my website

First off, welcome to my website! I have information about me on the [homepage]() and career information on my [CV]().
This website contains a hodgepodge of information including: useful links for CMS analysts, tutorials for using various 
particle physics tools, infomation about fun programming tools, and posts about my current research.

## The CMS Experiment

The Compact Muon Solenoid ([CMS](https://cms.cern)) experiment is particle detector at the Large Hadron Collider (LHC)
located at the European Organization for Nuclear Research ([CERN](https://home.cern)). CMS studies the fundamental building blocks of
matter by observing proton-proton collisions at the 27 km LHC. In 2017, the CMS experiment observed collisions every 
25 ns at center of mass energies of 13 TeV. In other words, CMS deals with high data rates, large amounts of data,
and large backgrounds. CMS analysts need to utilize a range of skills to handle these problems--including: building
custom electronics, working with machine learning algorithms, and using statistical software.



## Getting started

[GitHub Pages](https://pages.github.com) can automatically generate and serve the website for you.
Let's say you have a username/organisation `my-org` and project `my-proj`; if you locate Jekyll source under `docs` folder of master branch in your repo `github.com/my-org/my-proj`, the website will be served on `my-org.github.io/my-proj`.
The good thing about coupling your documentation with the source repo is, whenever you merge features with regarding content to master branch, it will also be published on the webpage instantly.

1. Just [download the source](https://github.com/aksakalli/jekyll-doc-theme/archive/gh-pages.zip) into your repo under `docs` folder.
2. Edit site settings in  `_config.yml` file according to your project. !!! `baseurl` should be your website's relative URI like `/my-proj` !!!
3. Replace `favicon.ico` and `img/logonav.png` with your own logo.

## Writing content

### Docs

Docs are [collections](https://jekyllrb.com/docs/collections/) of pages stored under `_docs` folder. To create a new page:

**1.** Create a new Markdown as `_docs/my-page.md` and write [front matter](https://jekyllrb.com/docs/frontmatter/) & content such as:

```
---
title: My Page
permalink: /docs/my-page/
---

Hello World!
```

**2.** Add the pagename to `_data/docs.yml` file in order to list in docs navigation panel:

```
- title: My Group Title
  docs:
  - my-page
```

### Blog posts

Add a new Markdown file such as `2017-05-09-my-post.md` and write the content similar to other post examples.

### Pages

The homepage is located under `index.html` file. You can change the content or design completely different welcome page for your taste. (You can use [bootstrap components](http://getbootstrap.com/components/))

In order to add a new page, create a new `.html` or `.md` (markdown) file under root directory and link it in `_includes/topnav.html`.
