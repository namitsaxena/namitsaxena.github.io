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
- Rename this to: <youraccountname>.github.io
- Update _config.yml (master branch is default)
  1. Update Name, title, etc
  2. baseUrl needs to be blank("") if this is the default domain
- Update CNAME to your domain name if you have one, or blank this out
- Once published, you can access the site on <youraccountname>.github.io (or your domain of you configured it)