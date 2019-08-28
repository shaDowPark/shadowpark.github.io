---
layout: post
title:  "Minikube with kvm"
date:   2019-07-17 08:59:34 +0200
categories: kubernetes vm container cloud native
---

In this article, i show you how to start minikube with another vm than VirtualBox. Here i use KVM (Kernel-based Virtual Macine).

# Requirement before start
----
You need to have already install docker on your machine.

# Kubectl (optional if you have already installed it)
----
In the first place, you need to install Kubectl, if it's not done yet.

{% highlight shell %}
sudo apt-get update && sudo apt-get install -y apt-transport-https
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubectl
{% endhighlight %}

test if the installation is ok 

{% highlight shell %}
kubectl version
{% endhighlight %}

you can enable shell autocompletion for kubectl by add this line a the end of you profile file, in my case i use zsh :

{% highlight shell %}
source <(kubectl completion zsh)
{% endhighlight %}

then you have to generated the autocompletion script

{% highlight shell %}
kubectl completion zsh
{% endhighlight %}

and to finish you need to reboot your shell process.

# Minikube
----

## Installation

To install minikube you need to download the last minikube release binary

{% highlight shell %}
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 \
  && chmod +x minikube
{% endhighlight %}

add it to your path 

{% highlight shell %}
sudo cp minikube /usr/local/bin && rm minikube
{% endhighlight %}

test installation is ok 

{% highlight shell %}
minikube version
{% endhighlight %}

by default minikube will look for VirtualBox when it start.
You want to tell minikube to use KVM instead, for that you need to install the vm manager 
Qemu and the API libvirt use by Qemu and KVM to communicate.

## Install vm manager Qemu

{% highlight shell %}
sudo apt install libvirt-bin qemu-kvm
sudo usermod -a -G libvirt $(whoami)
newgrp libvirt
{% endhighlight %}

now you need to install Minikube's kvm2 driver  which replaces the current kvm driver

{% highlight shell %}
curl -Lo docker-machine-driver-kvm2 https://storage.googleapis.com/minikube/releases/latest/docker-machine-driver-kvm2 \
&& chmod +x docker-machine-driver-kvm2 \
&& sudo cp docker-machine-driver-kvm2 /usr/local/bin/ \
&& rm docker-machine-driver-kvm2
{% endhighlight %}

## Start with kvm2 driver

now start Minikube 

{% highlight shell %}
minikube start --vm-driver kvm2
{% endhighlight %}

or 

{% highlight shell %}
minikube start --v=7
{% endhighlight %}

you get this 

![screen-start-minikube]({{ site.url }}/assets/screenshots/20190718_1547_screenshot.png)


## Test 

To test the smooth running of Minikube, use these commands

{% highlight shell %}
minikube status
{% endhighlight %}

and get this

![screen-start-minikube]({{ site.url }}/assets/screenshots/20190718_1607_screenshot.png)

connect to minikube with ssh

{% highlight shell %}
minikube ssh
{% endhighlight %}

![screen-start-minikube]({{ site.url }}/assets/screenshots/20190718_1611_screenshot.png)


it's done you have installed minikube on ubuntu and it run on top of kvm.


