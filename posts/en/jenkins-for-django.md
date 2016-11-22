Title: How to configure Jenkins for Django multibranch project
Date: 2016-11-25
Category: Django
Tags: django, jenkins, devops, linux, continuous integration, CI
Slug: django-jenkins
Image: /images/django.jpg
Lang: en

One day we (our work team) where tired to launch tests manually before every push and wait until
eveything passed in parrent branch then merge it to master, and do the same thing again. To simplify
this process and exclude the situations when somebody is forgeting about tests in a hurry, we
decided that we probably need to automate this process.

The choise fell on Jenkins because we where searching for free, Open Source and self-hosted
soulution that can be esily integrated with in any third party app including Django integration.

So here is the short tutorial how to get everything to work and save your time.

## Installation

In our case most of servers working on Ubuntu/Debian so this tutorial oriented for this two systems.

Recommended requirements:

- __OS__: Debian 8 or Ubuntu 16.04
- __CPU__: 2 cores
- __RAM__: 1-2GB

[DigitalOcean](https://m.do.co/c/6463f665f8bc){:rel=nofollow} droplet for $20/mo will ideally pass
your needs.

First we need to create user for our CI future CI server:

```
:::bash
#!/bin/bash
adduser jenkins
su jenkins
```
Then add necessary repositories and instal Jenkins:

```
:::bash
#!/bin/bash
# Same for ubuntu and for debian

wget -q -O - https://pkg.jenkins.io/debian/jenkins-ci.org.key | sudo apt-key add -
sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
sudo apt-get update
sudo apt-get install jenkins
sudo nano /etc/default/jenkins # Editing config
```

Now you need to set variable `JENKINS_HOME=/home/$NAME` in config file `/etc/default/jenkins`.

Now we can just run it:

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

After Jenkins was run you can access it via `<ip-address>:8080` for me it will be
`127.0.0.1:8080`.

![Fresh installation of Jenkins](/images/jenkins/jenkins-password.png)

You need to enter password which can be get by typing the following:

```
:::bash
#!/bin/bash
cat /home/jenkins/secrets/initialAdminPassword
a3cc46430ff249df9b1b4ce0ff90dcba # Password example that you would get

```

Next step, you will be able to choose plugins what you wan't to install I would recommend to choose
"Install proposed plugins" as a basic pack and then we will install manually all the other things.

![Installation of suggested plugins](/images/jenkins/jenkins-install-suggested-plugins.png)

Ok, plugins installed now it's time to create our first user:

![User creation process in jenkins](/images/jenkins/jenkins-create-user.png)

That's it! Installation finished, now we can configure our Jenkins instnace.

![Jenkins is ready to use](/images/jenkins/jenkins-ready-to-use.png)

## Configuring Jenkins

### Installation of necessary plugins

__Before__ creation any jobs we need to install plugins because we need specific project type
support.

First, you need to get to "Manage Jenkins" section:

![Jenkins is ready to use](/images/jenkins/jenkins-manage.png)

And then to "Manage plugins":
![Jenkins is ready to use](/images/jenkins/jenkins-manage-plugins.png)

Here you need to go to "Available" tab and search & install for following plugins:

- __Multi-Branch Project Plugin__ --- allows you to process multiple branches in one time.
- __Cobertura Plugin__ --- code coverage reports plugin.
- __JUnit__ _&nbsp;(if not installed)_ --- required for publishing tests results.
- _HipChat Plugin (optional)_ --- notify your team if something breaks in the HipChat.

![Plugins installation | Jenkins](/images/jenkins/jenkins-install-plugins.png)

### Setting up new project

Go to Jenkins start page and press __"Create new jobs"__ then enter name of your project, and choose
__"Freestyle multibranch project"__ type.

![Freestyle multibranch project | Jenkins](/images/jenkins/jenkins-new-project.png)

At first, we will need to configure VCS for our project, go to section "Branch sources" and enter
your project URL.

![Configure VCS for new project | Jenkins](/images/jenkins/jenkins-vcs-configuration.png)

Also, you will need to enter your credentials to access code, press "Add" - "Jenkins" and enter your
GitHub username and password or, choose other credentials type.

![Adding new credentials | Jenkins](/images/jenkins/jenkins-access.png)

In "branch sources" advanced settings you can enter branches which you need to test.

![Adding branch settings](/images/jenkins/jenkins-advanced-vcs-configuration.png)

Now we need to configure build step --- go to __"Build Configuration"__ section click __"Add build
step"__, and choose __"Execute shell"__ from the dropdown menu.

![Adding branch settings](/images/jenkins/jenkins-build-setup.png)

In the text area that appears we need to add following code:

```
:::bash

cd $WORKSPACE

# Virtualenv setup
virtualenv env --python=/usr/bin/python3.5
env/bin/pip install -r requirements_ci.txt

# You can add any custom tasks here, or write it into script and place to VCS.
# .....

export DJANGO_SETTINGS_MODULE=porjectname.settings_ci
env/bin/python3.5 ./manage.py collectstatic --noinput
env/bin/python3.5 ./manage.py jenkins --noinput --enable-coverage --parallel 2
```

## Django project configuration

### Requirements

To avoid unnecessary packages in main project virtualenv we will need to create `requiements_ci.txt`
file that extends your basic project requirements list.

```
-r requirements.txt
django-jenkins
coverage
pyflakes
tblib
```

### Separated settings for CI

We will need to create separate settings file named `settings_ci.py` with the following code:

```
:::python

import string
import random
from .settings import *


def id_generator(size=10, chars=string.ascii_uppercase + string.digits):
    """
    Generates random name for database.
    """
    return ''.join(random.choice(chars) for _ in range(size))


DEBUG = True

INTERNAL_IPS = ['127.0.0.1', 'localhost', 'example.com']
ALLOWED_HOSTS = INTERNAL_IPS

user = 'postgres'
password = 'postgres'
host = '127.0.0.1'
port = '5432'
# Random name to avoid errors during multiple branches parallel testing
db = id_generator()


DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql_psycopg2',
        'NAME': db,
        'USER': user,
        'PASSWORD': password,
        'HOST': host,
        'PORT': port,
    }
}


INSTALLED_APPS += [
    'django_jenkins',
]

JENKINS_TASKS = (
    'django_jenkins.tasks.run_pep8',
    'django_jenkins.tasks.run_pyflakes',
)
```

This is basic setup, for more information you can read
[django-jenkins documentation](https://github.com/kmmbvnr/django-jenkins/blob/master/README.rst){:rel=nofollow}.

## Manage final result

## Extras

### HipChat integration

## Sources

1. [Installing Jenkins on Ubuntu &mdash; Jenkins &mdash; Jenkins Wiki](https://wiki.jenkins-ci.org/display/JENKINS/Installing+Jenkins+on+Ubuntu){:rel=nofollow}.
2. [Continious Integration Testing](http://www.slideshare.net/kevincreedharvey/continuous-integration-testing-django-ap){:rel=nofollow}.
3. [jenkins  &mdash; Change JENKINS_HOME on Red Hat Linux?  &mdash; Stack Overflow](http://stackoverflow.com/questions/21839538/change-jenkins-home-on-red-hat-linux){:rel=nofollow}.

