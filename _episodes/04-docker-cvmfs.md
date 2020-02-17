---
title: "Accessing CVMFS from Docker locally"
teaching: 10
exercises: 15
questions:
- "How can I access CVMFS from my computer?"
- "How can I access CVMFS from Docker?"
objectives:
- "Be aware of CVMFS with Docker options."
- "Successfully mount CVMFS via a privileged container."
keypoints:
- "You can install CVMFS on your local computer."
- "The `cvmfs-automounter` allows you to provide CVMFS to other containers."
- "Privileged containers can be dangerous."
---

In order to use CVMFS via Docker, a couple of extra steps need to be taken.
There are different approaches:

1. Installing CVMFS on your computer locally and mounting it from the container.
1. Mounting CVMFS via another container and providing it to the analysis container.
1. Mounting CVMFS from the analysis container.

We will go through these options in the following.

> ## This is were things get ugly
>
> Unfortunately, all the options below have some caveats, and they might not
> even work on your computer. At the moment, no clear recommendations can be
> given. Try for yourself which option works best for you. However, it is not
> essential for this lessons that either of these options work.
>
{: .callout}

## Installing CVMFS on a local computer

CVMFS can be installed locally on your computer. Packages and installation
packages are provided on the [CVMFS Downloads page][cvmfs-download]. In the
interest of time, we will not install CVMFS now, but instead use the second
option above in the following. If you would like to install CVMFS on your
computer, make sure to read the
[CVMFS Client Quick Start Guide][cvmfs-quickstart].
Please also have a look at the
[CVMFS with Docker documentation][cvmfs-docker-docs]
to avoid common pitfalls when running Linux on your computer and trying to
bind mount CVMFS from the host. This is not necessary when running on a Mac.
However, you need to go to *Docker Settings* -> *Resources* -> *File Sharing*
and add `/cvmfs` to enable bind mounting.

![`/cvmfs` needs to be accessible to Docker for bind mounting](../fig/docker_file_sharing.png)

To run your analysis container and give it access to the `/cvmfs` mount, run
the following command (mind that `--rm` deletes the container after exiting):

~~~
docker run --rm -it -v /cvmfs:/cvmfs gitlab-registry.cern.ch/clange/cc7-cms /bin/bash
~~~
{: .language-bash}

## Using the `cvmfs-automounter`

The first option needed CVMFS to be installed on your computer. Using the
`cvmfs-automounter` is effectively mimicking what is done on GitLab. First, a
container, the `cvmfs-automounter`, is started that mounts CVMFS, and then
this container provides the CVMFS mount to other containers. If you are
running Linux, the following command should work. On a Mac, however, this will not work:

~~~
docker run -d --name cvmfs --pid=host --user 0 --privileged --restart always -v /shared-mounts:/cvmfsmounts:rshared gitlab-registry.cern.ch/vcs/cvmfs-automounter:master
~~~
{: .language-bash}

This container is running as a daemon (`-d`), but you can still see it via
`docker ps` and also kill it using `docker kill cvmfs`.

~~~
docker run -v /shared-mounts/cvmfs:/cvmfs:rslave -v $(pwd):$(pwd) -w $(pwd) --name ${CI_PROJECT_NAME} ${FROM} /bin/bash ./.gitlab/build.sh
~~~
{: .language-bash}

## Mounting CVMFS from the analysis container

~~~
docker run --rm --cap-add SYS_ADMIN --device /dev/fuse -it gitlab-registry.cern.ch/clange/cc7-cmssw-cvmfs:latest bash
~~~
{: .language-bash}

If you get an error similar to:

~~~
/bin/sh: error while loading shared libraries: libtinfo.so.5: failed to map segment from shared object: Permission denied
~~~
{: .output}

you need to turn off SElinux security policy enforcing:

~~~
sudo setenforce 0
~~~
{: .language-bash}

This can be changed permanently by editing `/etc/selinux/config`, setting `SELINUX` to `permissive` or `disabled`. Mind, however, that there are certain security issues with disabling SElinux security policies as well as running privileged containers.

- Using the cvmfs-automounter
- Special permissions (privileged)
- Running the build script in Docker with mounted CVMFS (point out both local CVMFS mount and automounter options)

{% include links.md %}

