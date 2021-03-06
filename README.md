# github pages based on [poole/lanyon](https://github.com/poole/lanyon)

## How To
* Fork https://github.com/poole/lanyon
* Rename to your username: namitsaxena.github.io
  * If you want to make this private, you will need to [duplicate](https://docs.github.com/en/github/creating-cloning-and-archiving-repositories/duplicating-a-repository) since forked repos can't be made private  
* Update master branch (master branch - unlike the instruction in poole)
  * _config.yml
  * CNAME
* If you want to keep this separate then update the baseUrl in config file above
  * baseurl: https://namitsaxena.github.io/poole/ (or as /poole - haven't tried it yet)
    * you need to fork and update [poole](https://github.com/poole/poole) for this setup to work (not needed for this setup!!!)
    * ex: https://github.com/s311354/Louis.github.io/blob/master/_config.yml#L13

## Notes
* Any files at the root level - (with script(~) or .md) show up in sidebar, by default (this behavior changed here to allow tigher control but please refer to the original _includes/sidebar code)
* All md files also get processed (as expected). Will show up as processed if accessed without extension. With extension show up in raw (with markdown) format
  ** if there are errors then files don’t show up processed
* html files can be accessed as it is since they don’t need to be processed
* certain files like index-blog.html are just accessed via http://pages.namitsaxena.com/index-blog/ (i.e. without .html)
  * plain md files are also accessed without extension
