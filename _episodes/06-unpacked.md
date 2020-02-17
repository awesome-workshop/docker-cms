---
title: "Using unpacked.cern.ch"
teaching: 10
exercises: 5
questions:
- "What is `unpacked.cern.ch`?"
- "How can I use `unpacked.cern.ch`?"
objectives:
- "Understand how your images can be put on `unpacked.cern.ch`"
keypoints:
- "The `unpacked.cern.ch` CVMFS area provides a very fast way of distributing unpacked docker images for access via Singularity."
---
As was pointed out in the previous episode, Singularity uses *unpacked* Docker
images. These are by default unpacked into the current working directory,
and the path can be changed by setting the `SINGULARITY_CACHEDIR` variable.

The EP-SFT group, provides a service that unpacks Docker images and makes them
available via a dedicated CVMFS area. In the following, you will learn how to
add your images to this area. Once you have your image(s) added to this area,
these images will be automatically synchronised within a few minutes whenever
you create a new version of the image.

## Exploring the CVMFS `unpacked.cern.ch` area

The *unpacked* area is a directory structure within CVMFS:

~~~
ls /cvmfs/unpacked.cern.ch/
~~~
{: .language-bash}

~~~
gitlab-registry.cern.ch  registry.hub.docker.com
~~~
{: .output}

You can see the full directory structure of an image:

~~~
ls /cvmfs/unpacked.cern.ch/gitlab-registry.cern.ch/clange/jetmetanalysis:latest
~~~
{: .language-bash}

~~~
afs  boot    cvmfs  environment  etc   lib    lost+found  mnt  pool  root  selinux      srv  tmp  var
bin  builds  dev    eos          home  lib64  media       opt  proc  sbin  singularity  sys  usr
~~~
{: .output}

This can be useful for investigating some internal details of the image.

As mentioned above, the images are synchronised with the respective registry.
However, you don't get to know when the synchronisation happened, but there
is an easy way to check by looking at the timestamp of the image directory:

~~~
ls -l /cvmfs/unpacked.cern.ch/gitlab-registry.cern.ch/clange/jetmetanalysis:latest
~~~
{: .language-bash}

~~~
lrwxrwxrwx. 1 cvmfs cvmfs 79 Feb 17 11:01 /cvmfs/unpacked.cern.ch/gitlab-registry.cern.ch/clange/jetmetanalysis:latest -> ../../.flat/c4/c4c848905d602f858e62b538114ab168c41c380837c05705e067868e6617a714
{: .output}

In the example given here, the image has last been updated on February 17th at 11:01.

## Adding to the CVMFS `unpacked.cern.ch` area

You can add your image to the `unpacked.cern.ch` area by making a merge
request to the [unpacked sync repository][unpacked-sync]. In this repository
there is a file called [`recipe.yaml`][unpacked-sync-recipe], to which you
simply have to add a line with your full image name (including registry)
prepending `https://`:

~~~
- https://gitlab-registry.cern.ch/clange/jetmetanalysis:latest
~~~
{: .language-yaml}

As of 14th February 2020, it is also possible to use wildcards for the
tags, i.e. you can simply add

~~~
- https://gitlab-registry.cern.ch/clange/jetmetanalysis:*
~~~
{: .language-yaml}

and whenever you build an image with a new tag it will be synchronised.

## Running Singularity using the `unpacked.cern.ch` area

Running Singularity using the `unpacked.cern.ch` area is done using the
same commands as listed in the previous episode with the only difference
that instead of providing a `docker://` image name to Singularity,
you provide the path in `/cvmfs/unpacked.cern.ch`:

~~~
singularity shell -B /afs -B /eos -B /cvmfs /cvmfs/unpacked.cern.ch/gitlab-registry.cern.ch/clange/jetmetanalysis:latest
~~~
{: .language-bash}

Now you should be in an interactive shell almost immediately without any
image pulling or unpacking. One important thing to note is that for most
CMS images the default username is `cmsusr`, and if you compiled your
analysis code in the container, it will by default reside in
`/home/cmsusr`:

~~~
Singularity> cd /home/cmsusr/CMSSW_5_3_32/src/
Singularity> source /cvmfs/cms.cern.ch/cmsset_default.sh
Singularity> cmsenv
~~~
{: .language-bash}


{% include links.md %}

