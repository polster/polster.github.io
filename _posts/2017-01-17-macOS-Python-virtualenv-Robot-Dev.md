---
layout: post
title:  "Python - Virtualenv for Robot Test Development on macOS"
date:   2017-01-17 22:55:10 +0100
---

## Introduction

* This blog entry summarizes an easy approach how to setup a virtual env for Robot Test development
* How to do it with homebrew on macOS instructions can be found everywhere in the wild, so motivation was to summarize the needed steps without

## Setup without homebrew

### direnv

{% highlight bash linenos %}
# Download the latest binary
curl -O https://github.com/direnv/direnv/releases/download/v2.4.0/direnv.darwin-amd64

# Create user bin dir and move the binary
mkdir ~/bin/direnv
mv direnv.darwin-amd64 ~/bin/direnv

# Make the direnv binary executable
chmod +x ~/bin/direnv

# Ensure the user bin dir is on the path (~/.profile or ~/.zshrc)
export PATH=~/bin:$PATH

# Add the following line if on bash (~/.bashrc)
eval "$(direnv hook bash)"

# Add the following line if on zsh (~/.zshrc)
eval "$(direnv hook zsh)"

# Star a new terminal session or source your bash/zsh env
{% endhighlight %}

### Virtualenv

{% highlight bash linenos %}
# Ensure virtualenv is installed
pip install virtualenv

# cd into your project folder and create a new direnv config file
echo "# Python
layout python
export PYTHONPATH=$VIRTUAL_ENV/lib/python2.7/site-packages/:$PYTHONPATH" > .envrc

# As requested by the direnv error message that should appear, run the following
direnv allow

# If everything worked, executing the following command should print the path to the python executable within this project
which python

{% endhighlight %}

### Robot Framework

{% highlight bash linenos %}
# Create another dot file containing the Robot Framework packages to be present
echo "pip install robotframework

pip install robotframework-sshlibrary
pip install robotframework-requests
pip install robotframework-selenium2library
pip install robotframework-extendedselenium2library" > .robotenv

# Make the file executable
chmod u+x .robotenv

# Install the defined packages
./.robotenv
{% endhighlight %}

## Setup with homebrew

See the [Robot Env Script](https://github.com/polster/dev-env-scripts/blob/master/robot-dev-env-setup.sh)
