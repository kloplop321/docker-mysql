# Table of Contents

- [Introduction](#introduction)
- [Contributing](#contributing)
- [Changelog](Changelog.md)
- [Reporting Issues](#reporting-issues)
- [Installation](#installation)
- [Quick Start](#quick-start)
- [Data Store](#data-store)
- [Creating User and Database at Launch](#creating-user-and-database-at-launch)
- [Shell Access](#shell-access)
- [Upgrading](#upgrading)

# Introduction

Dockerfile to build a MySQL container image which can be linked to other containers.

This also allows specifying a .sql file to use on import with `DB_IMPORT`

# Contributing

If you find this image useful here's how you can help:

- Send a Pull Request with your awesome new features and bug fixes
- Help new users with [Issues](https://github.com/sameersbn/docker-mysql/issues) they may encounter
- Send me a tip on [Gittip](https://gittip.com/sameersbn/) or using Bitcoin at **16rDxVqJPyYAFYPLduTaSiwe7ZiY1hHqKM**

# Reporting Issues

Docker is a relatively new project and is active being developed and tested by a thriving community of developers and testers and every release of docker features many enhancements and bugfixes.

Given the nature of the development and release cycle it is very important that you have the latest version of docker installed because any issue that you encounter might have already been fixed with a newer docker release.

For ubuntu users I suggest [installing docker](https://docs.docker.com/installation/ubuntulinux/) using docker's own package repository since the version of docker packaged in the ubuntu repositories are a little dated.

Here is the shortform of the installation of an updated version of docker on ubuntu.

```bash
sudo apt-get purge docker.io
curl -s https://get.docker.io/ubuntu/ | sudo sh
sudo apt-get update
sudo apt-get install lxc-docker
```

Fedora and RHEL/CentOS users should try disabling selinux with `setenforce 0` and check if resolves the issue. If it does than there is not much that I can help you with. You can either stick with selinux disabled (not recommended by redhat) or switch to using ubuntu.

If using the latest docker version and/or disabling selinux does not fix the issue then please file a issue request on the [issues](https://github.com/sameersbn/docker-mysql/issues) page.

In your issue report please make sure you provide the following information:

- The host ditribution and release version.
- Output of the `docker version` command
- Output of the `docker info` command
- The `docker run` command you used to run the image (mask out the sensitive bits).

# Installation

Pull the latest version of the image from the docker index. This is the recommended method of installation as it is easier to update image in the future. These builds are performed by the **Docker Trusted Build** service.

```bash
docker pull sameersbn/mysql:latest
```

Alternately you can build the image yourself.

```bash
git clone https://github.com/sameersbn/docker-mysql.git
cd docker-mysql
docker build -t="$USER/mysql" .
```

# Quick Start

Run the mysql image

```bash
docker run -name mysql -d sameersbn/mysql:latest
```

You can access the mysql server as the root user using the following command:

```bash
docker run -it --rm --volumes-from=mysql sameersbn/mysql:latest mysql -uroot
```

# Data Store

You should mount a volume at `/var/lib/mysql`.

SELinux users are also required to change the security context of the mount point so that it plays nicely with selinux.

```bash
mkdir -p /opt/mysql/data
sudo chcon -Rt svirt_sandbox_file_t /opt/mysql/data
```

The updated run command looks like this.

```
docker run -name mysql -d \
  -v /opt/mysql/data:/var/lib/mysql sameersbn/mysql:latest
```

This will make sure that the data stored in the database is not lost when the image is stopped and started again.

# Creating User and Database at Launch

> **NOTE**
>
> For this feature to work the `debian-sys-maint` user needs to exist. This user is automatically created when the database is installed for the first time (firstrun).
>
> However if you were using this image before this feature was added, then it will not work as-is. You are required to create the `debian-sys-maint` user
>
>```bash
>docker run -it --rm --volumes-from=mysql sameersbn/mysql \
>  mysql -uroot -e "GRANT ALL PRIVILEGES on *.* TO 'debian-sys-maint'@'localhost' IDENTIFIED BY '' WITH GRANT OPTION;"
>```

To create a new database specify the database name in the `DB_NAME` variable. The following command creates a new database named *dbname*:

```bash
docker run --name mysql -d \
  -e 'DB_NAME=dbname' sameersbn/mysql:latest
```

To create a new user you should specify the `DB_USER` and `DB_PASS` variables.

```bash
docker run --name mysql -d \
  -e 'DB_USER=dbuser' -e 'DB_PASS=dbpass' -e 'DB_NAME=dbname' \
  sameersbn/mysql:latest
```

The above command will create a user *dbuser* with the password *dbpass* and will also create a database named *dbname*. The *dbuser* user will have full/remote access to the database.

**NOTE**
- If the `DB_NAME` is not specified, the user will not be created
- If the user/database user already exists no changes are be made
- If `DB_PASS` is not specified, an empty password will be set for the user

# Shell Access

For debugging and maintenance purposes you may want access the container shell. Since the container does not allow interactive login over the SSH protocol, you can use the [nsenter](http://man7.org/linux/man-pages/man1/nsenter.1.html) linux tool (part of the util-linux package) to access the container shell.

Some linux distros (e.g. ubuntu) use older versions of the util-linux which do not include the `nsenter` tool. To get around this @jpetazzo has created a nice docker image that allows you to install the `nsenter` utility and a helper script named `docker-enter` on these distros.

To install the nsenter tool on your host execute the following command.

```bash
docker run --rm -v /usr/local/bin:/target jpetazzo/nsenter
```

Now you can access the container shell using the command

```bash
sudo docker-enter mysql
```

For more information refer https://github.com/jpetazzo/nsenter

Another tool named `nsinit` can also be used for the same purpose. Please refer https://jpetazzo.github.io/2014/03/23/lxc-attach-nsinit-nsenter-docker-0-9/ for more information.

# Upgrading

To upgrade to newer releases, simply follow this 3 step upgrade procedure.

- **Step 1**: Stop the currently running image

```bash
docker stop mysql
```

- **Step 2**: Update the docker image.

```bash
docker pull sameersbn/mysql:latest
```

- **Step 3**: Start the image

```bash
docker run -name mysql -d [OPTIONS] sameersbn/mysql:latest
```
