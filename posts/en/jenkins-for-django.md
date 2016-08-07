Title: How to configure Jenkins for Django multibranch project
Date: 2016-08-7
Category: Django
Tags: django, jenkins, devops, linux, contintious integration, CI
Slug: django-jenkins
Image: /images/django.jpg
Lang: en

[TOC]

## Installation

### Configuring the system
- __OS__: Debian 8 or Ubuntu 16.04
- __CPU__: 2 cores
- __RAM__: 1-2GB

[DigitalOcean](https://m.do.co/c/6463f665f8bc) droplet for $20/mo will ideally pass your needs.

First we need to create user for our jenkins instance:

```
:::bash
#!/bin/bash
adduser jenkins
su jenkins
```
### Getting Jenkins

```
:::bash
#!/bin/bash
# Same for ubuntu and for debian

wget -q -O - https://pkg.jenkins.io/debian/jenkins-ci.org.key | sudo apt-key add -
sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
sudo apt-get update
sudo apt-get install jenkins
```

After successful installation you can launch your CI server:

```
:::bash
systemctl start jenkins

# You will need to enter password when asked
==== AUTHENTICATING FOR org.freedesktop.systemd1.manage-units ===
Authentication is required to start 'jenkins.service'.
Authenticating as: ,,, (jenkins)
Password: 
==== AUTHENTICATION COMPLETE ===
```

Or if you are systemd hater:

```
:::bash
sudo /etc/init.d/jenkins start
```

### Basic setup

After Jenkins was runned you can acces it via `<ip-address>:8080` for me it will be
`127.0.0.1:8080`.

![Fresh installation of jenkins](/images/jenkins/jenkins-fresh-install.png)

You need to enter password which can be get by typing:

```
:::bash
#!/bin/bash
cat /var/lib/jenkins/secrets/initialAdminPassword
a3cc46430ff249df9b1b4ce0ff90dcba # Password example that you would get

```

Next step, you will need to choose "Install proposed plugins" and wait until installation finished.

![Installation of suggested plugins](/images/jenkins/jenkins-install-suggested-plugins.png)

Then you need to create first user (as ussually is admin):

![User creation process in jenkins](/images/jenkins/jenkins-create-user.png)

Installation finished now we can configure it.
![Jenkins is ready to use](/images/jenkins/jenkins-ready-to-use.png)

## Configuring Jenkins


## Django project configuration

## Manage final result


## Extras

### Security notes

### HipChat integration

## Sources

1. [Installing Jenkins on Ubuntu &mdash; Jenkins &mdash; Jenkins Wiki](https://wiki.jenkins-ci.org/display/JENKINS/Installing+Jenkins+on+Ubuntu)
2. [Continious Integration Testing](http://www.slideshare.net/kevincreedharvey/continuous-integration-testing-django-ap)

