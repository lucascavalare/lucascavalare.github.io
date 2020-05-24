---
layout: post
title: Wharfee - 
tags: [macOS Catalina, Python-py, CLI, Wharfee, Docker]
image: https://user-images.githubusercontent.com/19478719/82759143-4f317600-9de3-11ea-8b60-673ee94e72c6.jpg
share-image: https://user-images.githubusercontent.com/19478719/82759143-4f317600-9de3-11ea-8b60-673ee94e72c6.jpg
#bigimg:
#  - "/img/Wharfee_logo.png"
---

I've updated to `macOS Catalina - 10.15.4` pretty while ago, and since then, I'm facing some problems with some software dependencies 
already installed on the system. Today, I got one and I'll share It because It can be the problem of someone else as well. 

As explained in an earlier post, using __Wharfee__ will makes your life easier when managing `Docker`. __Wharfee__ is an
interactive CLI tool to play with `Docker` and It fills up the `docker commands` as tab complete dough. 

On the `macOS Catalina - 10.15.4` when trying to run the `wharfee` I have the following error:
```
Traceback (most recent call last):
  File "/usr/local/bin/wharfee", line 5, in <module>
    from wharfee.main import cli
  File "/usr/local/lib/python3.7/site-packages/wharfee/main.py", line 24, in <module>
    from .client import DockerClient
  File "/usr/local/lib/python3.7/site-packages/wharfee/client.py", line 13, in <module>
    from docker import AutoVersionClient
ImportError: cannot import name 'AutoVersionClient' from 'docker' (/usr/local/lib/python3.7/site-packages/docker/__init__.py)
```

Researching for possible solutions, I found [Using new Python Docker client gives 'module' object has no attribute 'AutoVersionClient' #152](https://github.com/goldmann/docker-squash/issues/152) 
that was related to `docker squash`, however due to the same 'module' object attribute `AutoVersionClient`. 

Using [docker-py](https://github.com/docker/docker-py), a Python library to Docker Engine API https://docker-py.readthedocs.io/ would conflict 
with the `Wharfee` on the `from docker import DockerClient` and `from docker import AutoVersionClient` basically. 

The simply `pip3 uninstall docker-py` and `pip3 install docker-py` should solve the problem so far. The latest stable version of `docker-py`
is `1.10.6` on the https://pypi.org/project/docker-py/ project.

Also, to keep `pip3` up to date I've just ran `pip3 install pip --upgrade` to upgrade `pip3` to the latest stable version `pip-20.1.1`.

Now I'm able to use `wharfee` to help me managing `Docker` easily and interactively though:

<p align="center">
<img width="1000" height="70" 
src="https://user-images.githubusercontent.com/19478719/82759026-3ecccb80-9de2-11ea-92af-852424653ee8.png">
</p>
