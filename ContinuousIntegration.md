# Spectrecoin Continuous Integration

This document describes the whole Continuous Integration setup of Spectrecoin.

## General system setup

The whole CI system is running on [Amazon AWS](https://aws.amazon.com/) and is mostly based
on [Docker](https://www.docker.com/) respectively Docker images.

## Jenkins setup

At the moment there are two different types of EC2 instances used:
* A [c5d.large](https://aws.amazon.com/about-aws/whats-new/2018/05/introducing-amazon-ec2-c5d-instances/)
instance for the Jenkins master.
* Multiple [...](...) instances for the build slaves.

### Jenkins master

The Jenkins master instance is running on an c5d.large instance, which is running a
[Debian Stretch AMI](https://wiki.debian.org/Cloud/AmazonEC2Image/Stretch) as
operating system with Docker and [Docker-Compose](https://docs.docker.com/compose/)
installed.

To bring up everything smoothly and easy to update, we forked the GitHub project
_docker-jenkins-nginx-letsencrypt_ into
[spectrecoin/docker-jenkins-nginx-letsencrypt](https://github.com/spectrecoin/docker-jenkins-nginx-letsencrypt)
and customized it slightly to our needs:
* Updated hostname entries
* Using named volume for Jenkins home directory
* Expose port 50000 for build slaves
* Use dedicated path to store nginx content

### Jenkins build slaves

The Jenkins build slaves where initially setup using a ??? instance, which was running
also a Debian Stretch AMI. On this instance the following installations and modifications
took place:
* Activation of Stretch backports repository
* Installation of
  * ca-certificates
  * curl
  * gnupg2
  * software-properties-common
  * git
  * git-lfs
  * docker-ce
  * xfsprogs
  * openjdk-8-jdk
  * qemu-user-static
* Put _admin_ account into _docker_ group
* Create folder _/jenkins_ and set _admin:admin_ as owner and group

Afterwards an own Spectrecoin-Builder AMI was created from this instance.

To spawn build slave instances on demand we use the
[Amazon EC2 Plugin](https://wiki.jenkins.io/display/JENKINS/Amazon+EC2+Plugin)
and configured slave instances using the above mentioned AMI, instance type ???
and label _docker_. So with this setup Jenkins starts new build slave instances
if a buildjob requires build slaves with label _docker_ and shut them down after
a certain idle timeout.

### Updating the CI

Updating the master is a quite easy task.
* Shutdown docker containers:
  ```
  admin@ip-172-31-9-36:~$ cd docker-jenkins-nginx-letsencrypt/
  admin@ip-172-31-9-36:~/docker-jenkins-nginx-letsencrypt$ docker-compose down
  ```
* Update docker images:
  ```
  admin@ip-172-31-9-36:~/docker-jenkins-nginx-letsencrypt$ docker-compose pull
  ```
* Start new Docker containers:
  ```
  admin@ip-172-31-9-36:~/docker-jenkins-nginx-letsencrypt$ docker-compose up -d
  ```

## General build setup

The Spectrecoin Linux build is mostly based on Docker images. This enables the possibility,
to test the build itself easily on different base systems respectively different Linux
distributions. As default distribution _Debian Stretch_ is used.

Also, each Repository on [Spectrecoin GitHub organization](https://github.com/spectrecoin),
which contains a so called [Jenkinsfile](https://jenkins.io/doc/book/pipeline/jenkinsfile/)
will be automatically build. Details see below.

### Build using Docker - Spectrecoin Docker images

To take advantage of the Docker concept and to improve build speed, the build itself and
also the corresponding Docker images are split into a base- and a builder-part and an
consolidating final Spectrecoin build.

#### Spectre-Builder

One thing to speedup the build is to split these build steps into an own build job, which
build or setup the build system itself and so produce with every run the same result. So
these steps where separated into the creation of dedicated [_builder images_](https://github.com/spectrecoin/spectre-builder)
for various Linux distributions, which contain all _buildtime_ dependencies.

#### Spectre-Base

With the same reason as the _builder images_ from before, [_base images_](https://github.com/spectrecoin/spectre-base)
where created. These images contain all _runtime_ dependencies of Spectrecoin and they
are not required for the _build_ of Spectrecoin. They are the basement for our ready to run
[Docker images](https://hub.docker.com/u/spectreproject/dashboard/).

#### Spectre-Distribution & GitHub-Uploader

The [Spectre-Distribution](https://github.com/spectrecoin/spectre-distribution) and
[GitHub-Uploader](https://github.com/spectrecoin/github-uploader) repositories contains
Dockerfiles, which will _not_ have real Docker images as their result. In this case
Docker is used as the tool to create the release artifacts for the supported Distributions.

In fact the following happens:

With the GitHub-Uploader repository a helper Docker image is created, which contains the
very useful tool [github-release](https://github.com/aktau/github-release). This tool
respectively the GitHub-Uploader Docker image can be used, to _upload_ all kind of artifacts
to a release tag on a GitHub repository. Exactly this will happen during the Docker build of
the Spectre-Distribution repository: A temporary Docker image for each supported Linux
distribution is created, which extracts the Spectrecoin binaries from the formerly build
Spectrecoin Docker images. After that these binaries will be archived together and uploaded
to the corresponding release tag on the Spectrecoin release section.

### Jenkins Buildjobs

The build job configuration on Jenkins is mostly based on a so called _GitHub Organizsation Scanner_.
This job type is configured with the URL to the GitHub organization and scans all existing repositories.
On each repository each branch is checked for the existence of a _Jenkinsfile_ and if found, a dedicated
buildjob for this branch is created. So there is no administration necessary to setup buildjobs in case
a new branch is created on one of the Git repositories as Jenkins handels them by himself. The jobs will
of course be removed, if the corrsponding branch is removed like after a merge of the branch into
another one.
