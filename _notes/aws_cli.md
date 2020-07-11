---
layout: page
title: AWS CLI commands cheatsheet
tags: [aws, tech, cheatsheet]
---

## AWS CLI 
#### Configuration
* aws configure
* aws configure list
* aws configure get region

#### Current user
* aws sts get-caller-identity --output text 
* aws sts get-caller-identity --output text --query Arn

#### S3
* aws s3api list-buckets
* aws s3api list-buckets --query "Buckets[].Name"
