---
slug: create-a-python-virtualenv-on-debian-10
author:
  name: Linode
  email: docs@linode.com
description: This guide provides a brief introduction to Python virtual environments using the virtualenv tool on Debian 10.
keywords: ["python", "python virtual environment", "virtualenv", "debian 10"]
tags: ["python","debian"]
license: '[CC BY-ND 4.0](https://creativecommons.org/licenses/by-nd/4.0)'
modified: 2020-04-07
modified_by:
  name: Linode
published: 2017-08-13
image: Python_virtualenv_Deb10.png
title: How to Create a Python Virtual Environment on Debian 10
h1_title: Creating a Python Virtual Environment on Debian 10
external_resources:
- '[virtualenv Official Documentation](http://virtualenv.pypa.io/)'
audiences: ["beginner"]
languages: ["python"]
relations:
    platform:
        key: python-virtual-env
        keywords:
            - distribution: Debian 10
aliases: ['/development/python/create-a-python-virtualenv-on-debian-10/']
---

## What is a Python Virtual Environment?

A Python virtual environment is an isolated project space on your system that contains its own Python executable, packages, and modules. Your Python applications and projects often have their own specific dependencies. With a virtual environment you can manage each of your project's distinct dependencies without having them interfere with each other. You can use the [*virtualenv*](https://pypi.org/project/virtualenv/) tool to create a virtual environment on your system. This guide shows you how to use virtualenv to create and run a Python virtual environment on a Debian 10 Linode.

## Before You Begin

1.  Complete the [Getting Started](/docs/guides/getting-started/) and [Securing Your Server](/docs/guides/set-up-and-secure/) guides to prepare your system.

1.  Update your system:

        sudo apt-get update && sudo apt-get upgrade

    {{< note >}}
This guide is written for a non-root user. Commands that require elevated privileges are prefixed with `sudo`. If you're not familiar with the `sudo` command, you can check our [Users and Groups](/docs/guides/linux-users-and-groups/) guide.
    {{< /note >}}

## Create a Python Virtual Environment

{{< note >}}
By default, [Python 3.7.3](https://docs.python.org/3.6/whatsnew/3.6.html) and [Python 2.7.16](https://docs.python.org/2.7/) are installed on Debian 10.
{{</ note >}}

1.  Install the virtualenv tool using your package manager:

        sudo apt install virtualenv

1.  Create a `python-environments` directory in your user's home directory and navigate to it:

        mkdir ~/python-environments && cd ~/python-environments

1. Create a Python virtual environment. By default, virtualenv attempts to use the Python 2.5 interpreter to create a new environment. Replace `env` with the name you would like to assign to your virtual environment.

        virtualenv env

    {{< disclosure-note "Change the Python Interpreter Version">}}
If you would like to create a virtual environment using Python3, use the `--python` option to pass the Python version you'd like to use.

    virtualenv --python=python3 env
    {{</ disclosure-note >}}

    The command creates a new directory with the name you assigned to your virtual environment. This directory contains all of the isolated files, packages, modules, and executables that is used by your new environment.

1.  Validate that your environment is installed with the version of Python that you expect:

        ls env/lib

    You should see your `env` environments Python version:

    {{< output >}}
python3.6
    {{</ output >}}

## Activate Your Virtual Environment

1. Activate the newly created virtual environment:

        source env/bin/activate

    The name of the working environment appears in parentheses after it's created.

      {{< output >}}
(env) example_user@hostname:~/python-environments$
      {{</ output >}}

      You can now begin installing Python packages and libraries that will remain isolated to your virtual environment.

## Deactivate a Virtual Environment

1. To deactivate an active virtual environment, issue the following command:

        deactivate

    Your virtual environment is deactivated and you should no longer see its name listed next to your command line's prompt

    {{< output >}}
example_user@hostname:~/python-environments$
    {{</ output >}}
