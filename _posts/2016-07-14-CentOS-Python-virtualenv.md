---
layout: post
title:  "Python - Virtualenv on CentOS"
date:   2016-07-14 22:55:10 +0100
---

## Introduction

* Having an older version of CentOS (e.g. 6.x) where the default Python version 2.6 cannot be replaced due to OS specific dependencies may require to install a newer version of Python that shall co-exist (or multiple versions)
* For such situation working with [virtualenv](https://virtualenv.pypa.io/en/stable/) could be a good solution
* The following example summarizes the installation of Python 2.7 needed by Robotframework

## Installing an individual Python version

{% highlight bash linenos %}
yum -y update
yum groupinstall -y 'development tools'

yum install -y zlib-devel bzip2-devel openssl-devel xz-libs wget

wget http://www.python.org/ftp/python/2.7.11/Python-2.7.11.tar.xz
xz -d Python-2.7.11.tar.xz
tar -xvf Python-2.7.11.tar

# Enter the directory
cd Python-2.7.11

# Run the configure
./configure --prefix=/usr/local

# compile and install it
make
make altinstall

# Checking Python version:
[user@host ~]# python2.7 -V
Python 2.7.11

# Install pip package manager
curl https://raw.githubusercontent.com/pypa/pip/master/contrib/get-pip.py | python2.7 -
{% endhighlight %}

## Installing and configuring the virtual env

{% highlight bash linenos %}
wget --no-check-certificate https://pypi.python.org/packages/source/s/setuptools/setuptools-1.4.2.tar.gz

# Extract the files:
tar -xvf setuptools-1.4.2.tar.gz
cd setuptools-1.4.2

# Install setuptools using the Python 2.7.11:
python2.7 setup.py install

# Install virtual env
pip2.7 install virtualenv
{% endhighlight %}

## Configure the virtual env

{% highlight bash linenos %}
# Create a new env which uses Python 2.7
virtualenv -p /usr/bin/python2.7 robot-env

# Start using the env
source robot-env/bin/activate

# Check the Python version
[user@host ~]# python -V
Python 2.7.11

# Install required pip packages

# Add the following line to ~/.bashrc in order to have the virtualenv being active at user login (optional)
source ~/robot-env/bin/activate
{% endhighlight %}
