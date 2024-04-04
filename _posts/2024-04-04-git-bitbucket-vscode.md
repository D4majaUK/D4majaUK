---
layout: post
title: Git - Bitbucket VSCode creds
categories:
  - git
---

# Problem

SSH not working

# Solution

Credential Manager

![First Computer](/Portfolio/images/CredMgr1.jpg)


Config

```
[user]
	name = <name>
	email = <email address>
[fetch]
	prune = false
[pull]
	rebase = false
[http]
	sslbackend = openssl
[credential]
	helper = wincred
[credential "<name in Credential Manager>"]
	usehttppath = true
[core]
	editor = \"C:\\.......\\AppData\\Local\\Programs\\Microsoft VS Code\\bin\\code\" --wait
```


Password

```
git config --global credential.helper wincred
```
