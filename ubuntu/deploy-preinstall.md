---
title: DEPLOY SETUP
description: n/a
weight: 1
---

{{% toc %}}

* Clone deploy repository from git
* Create ssh key for root user

```bash 
root: sudo su
root: <root user pass>
root: ssh-keygen
```

You should see files 2 files in your root ssh repo:

```
/root/.ssh/id_rsa
/root/.ssh/id_rsa.pub
```

Copy text from id_rsa.pub and past it to vagrant run up file:

Create new ssh keys and setup them on git, you can do it not by root:

```
albine: ssh-keygen
```

Install it on git service and copy key to vagrant ssh file directory:

<path/to/project>/deploy/vagrant/.ssh

* Replace all old git directories with you:

> Go to <path/to/project>/deploy/ 
> Find all maches with text `git_repo`
> Replace value with you git url


