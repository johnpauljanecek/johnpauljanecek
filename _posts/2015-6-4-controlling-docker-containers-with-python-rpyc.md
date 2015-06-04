---
layout: post
title: Controlling docker containers with python rpyc
---

In the post I am going to talk about controlling docker container with the python module rpyc. The main reason you will want to do this is to control multiple processes each running in an isolated virtual machine. I will then introduce the module I created rpyc_docker which makes it simpler to control multiple instances of docker. I use rpyc_docker primarily to run headless browsers, that way I can control multiple instances of a browser, each brower being a difference version if I desire. With some websites there is a problem with the browser freezing or crashing. When each browser runs in its own docker container this is not a problem, if the browser freezes or crashes it is isolated from the rest of your browser grid.

What is docker ?
----------------

Docker is built on top of [linux namespaces](http://man7.org/linux/man-pages/man7/namespaces.7.html), which are similar to bsd jails. Linux namespaces allow processes to be isolated in the linux kernel, each process with its own environment. Initially I attempted to use linux namespaces directly to isolate my processes, this is extremely complicated since each namespace requires its own file system and network to be configured.

Fortunately I discovered docker so I did not need to reinvent the wheel. Docker allows you to create extremely light wieght processes, each one running in its own customized linux environment. So your server could be running linux 14.04 but inside your docker container you could be running the latest version of redhat. It is best to check out the [docker website](https://www.docker.com/whatisdocker/) to see what all of the fuss is. With Docker it is extremely simple to customize new virtual machines and the processes inside them run with no extra overhead.

What is rpyc python ?
---------------------

Rpyc solves the problem on how to access python objects transparently inside a running docker container. The docker container is a running virtual machine, rpyc acts as a bridge to get around the isolation problem.

Rpyc is a remote procedure call library similar to [pyro](https://pypi.python.org/pypi/Pyro4) or [twisted spread](twistedmatrix.com/documents/current/core/howto/pb-usage.html). All three libraries allow you to perform rpc calls on python objects, and all three libraries are great libraries. Rpyc is designed for controlling machines which you own, and it is symetric. Which twisted spread and pyro are more designed for a client to server type model. In these two libraries it is assumed that you do not control the client, and possibly the client might be hostile.

Rpyc when running in classic mode makes the remote object appear as if it is locally on your machine. If you are unfamiliar with rpyc and classic mode read [Introduction to Classic RPyC](https://rpyc.readthedocs.org/en/latest/tutorial/tut1.html). With the remote object you can do things like monkey patching, if the object crashes you can get a remote traceback or even enter into a remote debugging session. This makes development a lot easier.

Rpyc also comes with other utilities like the ability to upload or download files to your remote machine.


Introducting Rpyc_docker
------------------------

Rpyc_docker is a python library which runs an rpyc server inside a docker container. There are instructions on how to setup and use rpyc_docker and rudimentary api documentation here which I am still working on [Rpyc_docker](https://johnpauljanecek.github.io/rpyc_docker/) and there is a [github repository](https://github.com/johnpauljanecek/rpyc_docker).

Rpyc_docker works well with the [page object](http://selenium-python.readthedocs.org/en/latest/page-objects.html) design pattern of testing and craping websites. For development purposes it is best to design with the browser visible and not running in a container. Later you can run multiple headless browsers each in its own docker container isolated and seperated.


