# Clementine Vagrant DevEnv

A Vagrantfile provisioned with Ansible that sets up a dev environment for working on Clementine projects.

This is only usable on OSX at the moment because the box uses NFS to mount the share folder.

It could be modified easily to be used with a windows machine by removing a couple lines or adding smb support. See https://github.com/blinkreaction/boot2docker-vagrant for a reference to how that might be accomplished.

Follow up to my attempt at using docker for setting up a dev environment for OSX for completing [**FreeCodeCamp**](http://www.freecodecamp.com/) [basejumps](http://www.freecodecamp.com/challenges/waypoint-get-set-for-basejumps):

https://github.com/dting/fcc-angular-fullstack-docker

## Why

Using vagrant allows for: 

* Separating the host os from the dev environment (Packages installed for the project don't end up on the host system). 
* Keeping packages required by the project are tracked in the provisioning files. 
* On-boarding a new member to the project with identical dev environments, up and running quickly.
* Mimicing production environments on local machines.

## Prerequisites

1. Vagrant (http://docs.vagrantup.com/v2/installation/index.html)
2. Virtualbox (https://www.virtualbox.org/wiki/Downloads)
3. Ansible (http://docs.ansible.com/ansible/intro_installation.html#installation)

If you have homebrew (http://brew.sh/) and homebrew cask (http://caskroom.io/):

    $ brew cask install virtualbox
    $ brew cask install virtualbox-extension-pack
    $ brew cask install vagrant
    $ brew install ansible

## Setup and Usage

Clone repository into a folder that will become your app's vagrant folder and change to that directory.

    $ git clone git@github.com:dting/clementine-vagrant-devenv.git myProject
    $ cd myProject

**_[ Danger! Make sure to be inside the new project directory before continuing! ]_**

Remove git files before you start.

    myProject$ find . | grep .git | xargs rm -rf

Bring up the vm's and wait for ansible to provision them.

    myProject$ vagrant up

This first vagrant up will take a fairly long time to complete. It provisions both machines (a dev/web app machine and a mongodb machine).
When this completes ssh into the web vm. When working on the project you should probably leave a terminal window up connected to this vm. If you need to install bower components or npm packages it should be done in this vm.

    myProject$ vagrant ssh web

You should see some banner text and a prompt:

    vagrant@web:/vagrant/project$

Run:

    vagrant@web:/vagrant/project$ npm install && bower install

On the host machine open an editor to edit `myProject/project/server.js`. Change:

    db.connect('mongodb://localhost:27017/clementinejs');

to:

    db.connect('mongodb://db:27017/clementinejs');

Back in the vm:

    vagrant@web:/vagrant/project$ gulp

On the host, navigate a browser to: [`192.168.111.222:3000`](http://192.168.111.222:3000).

## Git

At some point you will want to create a git repository with your code. It is up to you to decide if you want to include the Vagrantfile and provisioning directory along with your app.

    myProject$ git init
    myProject$ git commit -m "Initial commit."

or

    myProject/app$ git init
    myProject/app$ git commit -m "Initial commit."

If ssh-agent-forwarding is setup git can also be used inside the vm (see more about this later).

## Workflow

The editing of the files should be done on the host machine. The changes will be picked up by the vm and reflected via livereload to the host browsers pointed at the `http://<vm's ip (default:192.168.111.222)>:9000` when the `gulp` task is running in the vm.

## ssh-agent-forwarding

ssh-agent-forwarding is used so git can be used seemlessly on the vagrant web vm. You can read an in depth over article about it here:

http://www.unixwiz.net/techtips/ssh-agent-forwarding.html

On the host in `~/.ssh/config` add:

    Host 192.168.111.222
      ForwardAgent yes

Store the pass phrases on your keychain:

    $ ssh-add -K

From inside the web vm:

    vagrant@web:/vagrant/app$ ssh -T git@github.com

and you should see something like:

    The authenticity of host 'github.com (192.30.252.131)' can't be established.
    RSA key fingerprint is 16:27:ac:a5:76:28:2d:36:63:1b:56:4d:eb:df:a6:48.
    Are you sure you want to continue connecting (yes/no)? yes
    Warning: Permanently added 'github.com,192.30.252.131' (RSA) to the list of known hosts.
    Hi dting! You've successfully authenticated, but GitHub does not provide shell access.
