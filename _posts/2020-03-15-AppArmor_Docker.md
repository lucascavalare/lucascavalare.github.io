---
layout: post
title: How to disable apparmor to docker-default profile on Ubuntu/Debian?
tags: [Ubuntu/Debian, Linux Kernel, AppArmor, Docker]
image: https://lucascavalare.github.io/img/apparmor_docker/AppArmor_logo.png
share-image: https://lucascavalare.github.io/img/apparmor_docker/AppArmor_logo.png
#bigimg:
#  - "/img/AppArmor_logo.png"
---

__AppArmor__ ("Application Armor") is a Linux kernel security module that allows the system administrator to restrict programs' capabilities with per-program profiles. 
Profiles can allow capabilities like network access, raw socket access, and the permission to read, write, or execute files on matching paths.

Working on Docker binary to test new deployments to it I have faced some problems on how to disable `AppArmor` to `docker-default` profile.
It was a bit confusing because I did not see any `docker-default` profile into `/etc/apparmor.d/` where we usually have them as well. 
I have tried to `aa-complain` as usage: `usage: aa-complain [-h] [-d DIR] [--no-reload] program [program ...]`, however, the [-d DIR] parameter
was not working to `/usr/bin/docker` default installation. 

# Issue:
 ```
    To set a process to complain mode, use the command line tool
    'aa-complain'. To really tear down all profiles, run 'aa-teardown'."
    Removing test suite binaries
    Makefile:234: recipe for target 'test' failed
    make: *** [test] Error 1
 ``` 
 
 
# Check:
 ```
    $ root@consul-server:/etc/apparmor.d# aa-status
    apparmor module is loaded.
    1 profiles are loaded.
    1 profiles are in enforce mode.
       docker-default
    0 profiles are in complain mode.
    0 processes have profiles defined.
    0 processes are in enforce mode.
    0 processes are in complain mode.
    0 processes are unconfined but have a profile defined.
 ```

I found a related issue in a thread of the [Moby Project](https://github.com/moby/moby) [#33060](https://github.com/moby/moby/issues/33060#issuecomment-419363270).
The thread is about the same issue named to __Impossible to see contents of AppArmor Profile "docker-default"__. Hereto a `docker-default` file profile did solve it. 

# So how I deal with that ?

A generated profile based on a [template](https://raw.githubusercontent.com/moby/moby/master/profiles/apparmor/template.go) and be sure the profile is named `docker-default`.
Do a `copy-paste` to this configuration file in `/etc/apparmor.d/docker-default`. 

 ```
    #include <tunables/global>

    profile docker-default flags=(attach_disconnected,mediate_deleted) {

      #include <abstractions/base>

      ptrace peer=@{profile_name},

      network,
      capability,
      file,
      umount,

      deny @{PROC}/* w,  # deny write for all files directly in /proc (not in a subdir)
      # deny write to files not in /proc/<number>/** or /proc/sys/**
      deny @{PROC}/{[^1-9],[^1-9][^0-9],[^1-9s][^0-9y][^0-9s],[^1-9][^0-9][^0-9][^0-9]*}/** w,
      deny @{PROC}/sys/[^k]** w,  # deny /proc/sys except /proc/sys/k* (effectively /proc/sys/kernel)
      deny @{PROC}/sys/kernel/{?,??,[^s][^h][^m]**} w,  # deny everything except shm* in /proc/sys/kernel/
      deny @{PROC}/sysrq-trigger rwklx,
      deny @{PROC}/kcore rwklx,
      deny @{PROC}/mem rwklx,
      deny @{PROC}/kmem rwklx,

      deny mount,

      deny /sys/[^f]*/** wklx,
      deny /sys/f[^s]*/** wklx,
      deny /sys/fs/[^c]*/** wklx,
      deny /sys/fs/c[^g]*/** wklx,
      deny /sys/fs/cg[^r]*/** wklx,
      deny /sys/firmware/** rwklx,
      deny
    }
 ```

# RUN:

__P.S.__ If you are not `root`, make sure to use `sudo` within the command:
 ```
    $ root@consul-server:/etc/apparmor.d# aa-disable /etc/apparmor.d/docker-default
    Disabling /etc/apparmor.d/docker-default.
 ```
If you don't have `aa-disable` command installed. Run: `sudo apt-get install apparmor-utils`.

# CHECK:

 ```
    $ root@consul-server:/etc/apparmor.d# aa-status
    apparmor module is loaded.
    0 profiles are loaded.
    0 profiles are in enforce mode.
    0 profiles are in complain mode.
    0 processes have profiles defined.
    0 processes are in enforce mode.
    0 processes are in complain mode.
    0 processes are unconfined but have a profile defined.
 ```

Now you are able to proceed with whatever you are doing to solve the issue with `AppArmor`. Otherwise, make sure you set the profile back to enforce mode so forth.  
