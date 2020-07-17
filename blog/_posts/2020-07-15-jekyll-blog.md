---
layout: page
title: How to create a static blog using Jekyll and GitHub
categories: [tech]
tags: [tech, jekyll]
comments: true 
---
## Overview
A static blog can easily be hosted in GitHub. Here we will be using [Lanyon](https://github.com/poole/lanyon) theme which is a variant of [poole](https://github.com/poole/poole). This post won't be covering the basics of Jekyll but the resources given below provide a good overview of the same.

## Quick Start
To get up and running in two minutes, the following steps can be used:
- Fork https://github.com/poole/lanyon
- Rename this to: youraccountname.github.io
- Update _config.yml (master branch is default)
  1. Update Name, title, etc
  2. baseUrl needs to be blank("") if this is the default domain
- Update CNAME to your domain name if you have one, or blank this out
- Once published, you can access the site on youraccountname.github.io (or your domain of you configured it)
  
## How does this work
- All the references within _posts get processed
- all md files get processed and can be accessed without extension
- any html/md files at root level get added to the sidebar
- if there are any processing issues, github will notify you via email

## Customizations
1. Create a separate blog collection
2. Customize sidebar 
3. Add categories and tags indices
4. Add archive page
5. Enable commenting using Disqus
6. Add google analytics
7. Add gravatar pic to the sidebar
8. Add favicon 
9. Add social media buttons

## Tips
1. Simple markdown can be used to make blog/site entries. But if needed, a [prose.io](https://prose.io/) can be used as an online editor.
2. You can clone another repo in your github account (other than the default static page one) and use the baseUrl to access it. Example: [http://blog.namitsaxena.com/poole](http://blog.namitsaxena.com/poole/)

## Other Popular Themes 
- [Poole](https://github.com/poole/poole): provide a clear and concise foundational setup for any Jekyll site. It does so by furnishing a full vanilla Jekyll install with example templates, pages, posts, and styles.
  - [Poole/Lanyon](https://github.com/poole/lanyon): Lanyon is an unassuming Jekyll theme that places content first by tucking away navigation in a hidden drawer.
    - Example: [codinfox.github.io](https://codinfox.github.io/dev/2015/03/06/use-tags-and-categories-in-your-jekyll-based-github-pages/)
  - [Poole/Hyde](https://github.com/poole/hyde): Hyde is a brazen two-column Jekyll theme that pairs a prominent sidebar with uncomplicated content. 
    - Example: [longqian.me](https://longqian.me/2017/02/09/github-jekyll-tag/)
- [centrarium](https://github.com/bencentra/centrarium): A simple yet classy theme for your Jekyll website or blog.
  - Example [www.nikhita.dev](https://www.nikhita.dev/build-blog-using-github-jekyll)
- [Jekyll-Now](https://github.com/barryclark/jekyll-now): Simple static site generator that's perfect for GitHub hosted blogs 
- [beautiful-jekyll](https://github.com/daattali/beautiful-jekyll): ready-to-use template to help you create a beautiful website quickly. Perfect for personal sites, blogs, or simple project websites
  - Example [https://beautifuljekyll.com/](https://beautifuljekyll.com/)
- [pmarsceill.github.io](https://pmarsceill.github.io/just-the-docs/): Focus on writing good documentation

## References
- [Jekyll Official Documentation](https://jekyllrb.com/docs/)
- [Jekyll Introduction](http://jekyllbootstrap.com/lessons/jekyll-introduction.html)
- [Jekyll: Step by Step](https://jekyllrb.com/docs/step-by-step/01-setup/)
- [How to build a blog using Github, Jekyll and Lanyon by Nikhita Raghunath](https://www.nikhita.dev/build-blog-using-github-jekyll)
- [Use Tags and Categories in your Jekyll based Github Pages without plugins](https://codinfox.github.io/dev/2015/03/06/use-tags-and-categories-in-your-jekyll-based-github-pages/)
