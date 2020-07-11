---
layout: blog
title: Configuring Multiple SSH Accounts in GitHub
tags: [git, tech]
---

GitHub doesn't allow to use the same ssh key with multiple accounts. If you have two keys, then you need to configure your ssh config so that it automatically chooses the right key based on the url
1. Generate keys
    1. ssh-keygen -t rsa_2 -b 4096 -C "some.email@gmail.com"
        1. enter passphrase or leave blank
        2. enter a filename so that it doesn’t overwrite the default file
2. Add keys to ssh-agent
    1. ssh-add ~/.ssh/id_rsa
    2. ssh-add ~/.ssh/id_rsa_2
    3. Verify by running: ssh-add -l

3. configure your ssh config file
    ```
    Host anything.github.com
       HostName github.com
       PreferredAuthentications publickey
       AddKeysToAgent yes
       UseKeychain yes
       IdentityFile ~/.ssh/id_rsa_2
       IdentitiesOnly yes

    # * is default and applies to everything
    # so putting an exception for the above profile
    Host * !anything.github.com
       AddKeysToAgent yes
       UseKeychain yes
       IdentityFile ~/.ssh/id_rsa
    ```   
4. Verify your setup
	```
    | => ssh -T git@anything.github.com 
    Hi anything! You've successfully authenticated, but GitHub does not provide shell access.
    3. ________________________________________________________________________________
    | => ssh -T git@github.com 
    Hi yourname! You've successfully authenticated, but GitHub does not provide shell access.
    ```
5. Clone a repo
    1. you will need to use ssh (not https)
    2. Replace github.com with anything.github.com
        1. Ex: if shown in GitHub git@github.com:barryclark/jekyll-now.git, use
            1. git clone git@anything.github.com:barryclark/jekyll-now.git
                1. you can verify this by running
                    1. git remote show origin
                        1. Fetch URL: git@anything.github.com:barryclark/jekyll-now.git
