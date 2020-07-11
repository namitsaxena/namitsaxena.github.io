---
layout: page
title: Git Cheatsheet
categories: [tech]
tags: [git, tech, cheatsheet]
---

### squash commits
1. Choose the exact number of commits that you want to squash
```git rebase -i HEAD~4```
1. In the menu that shows up, supply 'r' (reword) command for the top line (also note that in git the lines are listed in reverse chronological order
1. For all other lines, supply 's' (squash) command, and save (:x) the menu  
1. The next menu would allow you to reword the top line commit, and the subsequent one would squash the remaining commits
1. force push the squashed commits (```git push --force```)  
