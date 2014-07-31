Title: My git setup
Date: 2012-02-01 10:20
Category: blog
Tags: ubuntu, git 

This is a quick overview on how I have setup my git repo on one of my servers.

The goal was to have a git repo that i was able to update from any where that I happened to be, that was not just stored on my laptop.


The server setup
----------------

Assuming the you already have setup all the SSH access you need then you just need to install git-core.

```sh
$ sudo apt-get install git-core
```


The client setup
-----------------

I am a small shell script that I use to create a new git repo on my git server.

```sh
#!/bin/bash
REPO="${1}"
GIT_SERVER="dansysadm.com"

if [[ "${REPO}" == "" ]];then
	echo "missing repo."
	exit 1
fi

new_repo=$( ssh ${GIT_SERVER} "[[ -e ~/git/${REPO} ]] && echo 'OK'" )
if [[ "${new_repo}" != "OK" ]];then
	ssh ${GIT_SERVER}  "mkdir -p ~/git/${REPO}"
	echo "NEW Repo directory setup"
fi	

new_git_repo=$( ssh ${GIT_SERVER}  "[[ -e ~/git/${REPO}/HEAD ]] && echo 'OK'" )
if [[ "${new_git_repo}" != "OK" ]];then
	ssh ${GIT_SERVER}  "cd ~/git/${REPO} && git --bare init"
	echo "NEW Git repo setup - ${GIT_SERVER}:~/git/${REPO}"
fi

if [[ ! -d ~/my_git ]];then
	mkdir -p ~/my_git
fi

if [[ ! -d ~/my_git/${REPO} ]];then
	echo "Cloning into ~/my_git/${REPO}"
	mkdir -p ~/my_git/${REPO}
    cd ~/my_git/${REPO}
    git init
    touch README
    git add README
    git commit -m 'first commit'
    git remote add origin ${GIT_SERVER}:~/git/${REPO}
    git push origin master
	echo "New local copy of the repo: ${GIT_SERVER}:~/git/${REPO} is setup. "
fi

cd ~/my_git/${REPO}

```

Creating a brand new repo
-------------------------

The below example is setting up a git repo on my repo server, then settings up my local system to point
at my git server as the origin master.

In this example the the new git repo is called example_repo'.

```sh
$ ~/bin/mygit example_repo
```

And the output that it generates...

```sh
NEW Repo directory setup
Initialized empty Git repository in /home/dannyla/git/example_repo/
NEW Git repo setup - dansysadm.com:~/git/example_repo
Cloning into ~/my_git/example_repo
Initialized empty Git repository in /home/danny/my_git/example_repo/.git/
[master (root-commit) 0e3343b] first commit
 0 files changed
 create mode 100644 README
Counting objects: 3, done.
Writing objects: 100% (3/3), 213 bytes, done.
Total 3 (delta 0), reused 0 (delta 0)
To dansysadm.com:~/git/example_repo
 * [new branch]      master -> master
New local copy of the repo: dansysadm.com:~/git/example_repo is setup. 
```

Making sure that its all setup.

```sh
$ cd ~/my_git/example_repo 
$ git log
commit 0e3343b7a778c9793b0ab9a5a489e5a01167e53b
Date:   Wed Feb 20 01:55:16 2013 +1100

    first commit
(END)

```

Next steps
----------

This is up to use https as well, so that i can easily push and pull via a http proxy
