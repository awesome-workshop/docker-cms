---
title: "Light-weight CMSSW containers"
teaching: 10
exercises: 0
questions:
- "How can I get a more light-weight CMSSW container?"
- "What are the caveats of using a light-weight CMSSW container?"
objectives:
- "Understand how the light-weight CMS containers can be used."
keypoints:
- "The light-weight CMS containers need to mount CVMFS."
- "Mounting CVMFS is done differently depending on the execution platform and container run-time."
---
In a similar way to [running CMSSW in GitLab][gitlab-cms-lesson], the
images containing only the base operating system (e.g. Scientific Linux 5/6
or CentOS 7/8) plus additionally required system packages can be used to run
CMSSW (and other related software). CMSSW needs to be mounted via CVMFS.
**This is the recommended way!**

## Adding analysis code to a light-weight container

Instead of using these containers only for compiling and
[running CMSSW][gitlab-cms-lesson-running], we can add our (compiled) code to
those images, building on top of them. The advantage in doing so is that you
will effectively be able to run your code in a version-controlled sandbox, in
a similar way as grid jobs are submitted and run. Adding your code on top of
the base image will only increase their size by a few Megabytes. CVMFS will
be mounted in the build step and also whenever the container is executed.
The important conceptual difference is that we do not use a *Dockerfile* to
build the image since that would not have CVMFS available, but instead we use
Docker *manually* as if it was installed on a local machine.

The way this is done is by requesting a **docker-privileged** GitLab runner.
With such a runner we can run Docker-in-Docker, which allows to manually
attach CVMFS to a container and run commands such as compiling analysis code
in this container. Compiling code will add an additional layer to the
container, which consists only of the effect of the commands run. After
exiting this container, we can tag this layer and push the container to the
container registry.

The *YAML* required looks as follows:

~~~
build_docker:
  only:
    - pushes
    - merge_requests
  tags:
    - docker-privileged
  image: docker:19.03.1
  services:
  # To obtain a Docker daemon, request a Docker-in-Docker service
  - docker:19.03.1-dind
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_BUILD_TOKEN $CI_REGISTRY
    # Need to start the automounter for CVMFS:
    - docker run -d --name cvmfs --pid=host --user 0 --privileged --restart always -v /cvmfsmounts:/cvmfsmounts:rshared gitlab-registry.cern.ch/vcs/cvmfs-automounter:master
  script:
    # ls /cvmfs/cms.cern.ch/ won't work, but from the container it will
    # If you want to automount CVMFS on a new docker container add the volume config /cvmfsmounts/cvmfs:/cvmfs:rslave
    - docker run -v /cvmfsmounts/cvmfs:/cvmfs:rslave -v $(pwd):$(pwd) -w $(pwd) --name ${CI_PROJECT_NAME} ${FROM} /bin/bash ./.gitlab/build.sh
    - SHA256=$(docker commit ${CI_PROJECT_NAME})
    - docker tag ${SHA256} ${TO}
    - docker push ${TO}
  variables:
    FROM: clelange/cc7-cms # Override the image specified in the Dockerfile
    TO: ${CI_REGISTRY_IMAGE}:${CI_COMMIT_SHORT_SHA}
    DOCKER_TLS_CERTDIR: "/certs"
~~~
{: .language-yaml}

This is pretty complicated, so let's break this into smaller pieces.

{% include links.md %}
