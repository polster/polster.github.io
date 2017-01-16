---
layout: post
title:  "Python - Virtualenv for Robot Test Development on macOS"
date:   2017-01-17 22:55:10 +0100
---

## Introduction

* This blog entry summarizes how to setup a virtual env for Robot Test development

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

{% endhighlight %}

### Virtualenv
