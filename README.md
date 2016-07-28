# Supported tags and respective `Dockerfile` links

-	[`latest` (*latest/Dockerfile*)](https://)

[![](https://badge.imagelayers.io/httpd:latest.svg)](https://imagelayers.io/?images=httpd:2.2.31,httpd:2.2.31-alpine,httpd:2.4.23,httpd:2.4.23-alpine) **TODO**

# What is JIRA Software?

JIRA is a proprietary issue tracking product, developed by Atlassian. It provides bug tracking, issue tracking, and project management functions. Please visit [official](https://confluence.atlassian.com/jirasoftwareserver071/jira-software-documentation-800706335.html) documentation for more information.
_JIRA Software_ is intended for use by software development teams and includes _JIRA Core_ and _JIRA Agile_.

> [wikipedia.org/wiki/Jira_(software)](https://en.wikipedia.org/wiki/Jira_(software))

# How to use this image.

This image is based on official [java:8 (JDK)](https://github.com/docker-library/docs/tree/master/java) and installs Atlassian JIRA Software 7.1.9. It is also enabled for use with reverse proxy by providing environment variables as explained further down in this README.
Reason why this Docker image has been established is that there are currently none official JIRA Software image from Atlassian and we wanted to do few tweaks of those unofficial images that exists on Docker Hub (see Credits below).

### Credits
We want to give credit to following Docker images that has been used as inspiration of this image:
- [atlassian/bitbucket-server](https://bitbucket.org/atlassian/docker-atlassian-bitbucket-server)
- [cptactionhank/docker-atlassian-jira-software](https://github.com/cptactionhank/docker-atlassian-jira-software)
- [ahaasler/docker-jira](https://github.com/ahaasler/docker-jira)

### Alt 1: Run with minimum config

```console
$ docker run -d -p 8080:8080 --name jira acntech/adop-jira
```

You are now ready to start configuration of JIRA (choosing license model and other initial configuration) by entering http://localhost:8080. We recommend that you look at logs (`docker logs jira -f`) while initial configuration is done to make sure everything is running smooth.

This will store the workspace in `/var/atlassian/application-data/jira`. All JIRA data lives in there - including plugins, configuration, attachments ++ (see [JIRA application home directory](https://confluence.atlassian.com/adminjiraserver071/jira-application-home-directory-802593036.html) ). You will probably want to make that a persistent volume (recommended)

### Alt 2: Configure persisting volume

```console
$ docker run -d -p 8080:8080 --name jira \
        -v "/your/home:/var/atlassian/application-data/jira" \
        acntech/adop-jira
```
This will store the JIRA data in `/your/home` on the host. Ensure that `/your/home` is accessible by the `jira` user in container (`jira` user - uid `1000`) or use [-u](https://docs.docker.com/engine/reference/run/#/user) `some_other_user` parameter with `docker run`.

### Alt 3: Configure reverse proxy

If you have a reverse proxy, such as [Nginx](https://confluence.atlassian.com/jirakb/integrating-jira-with-nginx-426115340.html) or [Apache HTTP Server](https://confluence.atlassian.com/kb/integrating-apache-http-server-reverse-proxy-with-jira-753894357.html) in front of your JIRA application you need to provide proxy settings:

```console
$ docker run -d -p 8080:8080 --name jira \
        -v "/your/home:/var/atlassian/application-data/jira" \
        -e "X_PROXY_NAME=example.com" \
        -e "X_PROXY_PORT=80" \
        -e "X_PROXY_SCHEME=http" \
        -e "X_PATH=/jira" \
        acntech/adop-jira
```

Environment Variables:
`X_PROXY_NAME`      : Sets the connector proxy name (in this case `example.com`)
`X_PROXY_PORT`      : Sets the connector proxy port (in this case `80`)
`X_PROXY_SCHEME`    : Sets the connector scheme (in this case `http`).
`X_PATH`            : Sets the context path (in this case `/jira` so you would access JIRA http://localhost:8080/jira).

_IMPORTANT_! This configuration will be only written to `${JIRA_INSTALL_DIR}/conf/server.xml` file once, when one or more of env variables are provided. Next time you stop/start container these parameters will be ignored.

You will also need to configure reverse proxy, _example_ of such configuration for _Nginx_ (which is running at same [Docker host and network](https://docs.docker.com/engine/userguide/networking/dockernetworks/) as JIRA) is:
```
server {
    listen                                  80;
    server_name                             example.com;
    location /jira {
        proxy_set_header X-Forwarded-Host   $host;
        proxy_set_header X-Forwarded-Server $host;
        proxy_set_header X-Forwarded-For    $proxy_add_x_forwarded_for;
        proxy_pass                          http://jira-software:8080;
        proxy_redirect                      off;
    }
}
```
### Alt 4: Configure custom memory and plugin timeout properties

```console
$ docker run -d -p 8080:8080 --name jira \
      -v "/your/home:/var/atlassian/application-data/jira" \
      -e "X_PROXY_NAME=example.com" \
      -e "X_PROXY_PORT=80" \
      -e "X_PROXY_SCHEME=http" \
      -e "X_PATH=/jira" \
      -e "CATALINA_OPTS=-Xms1024m -Xmx2048m -Datlassian.plugins.enable.wait=300" \
      acntech/adop-jira
```

Catalina properties:
`Xms` : JVM Minimum Memory (in this case 1 GB). [More information](https://confluence.atlassian.com/adminjiraserver070/increasing-jira-application-memory-749383419.html)
`Xmx` : JVM Maximum Memory (in this case 2 GB). [More information](https://confluence.atlassian.com/adminjiraserver070/increasing-jira-application-memory-749383419.html)
`atlassian.plugins.enable.wait` : Time in seconds JIRA waits for plugins to load eg. 300. [More information](https://confluence.atlassian.com/display/JIRAKB/JIRA+applications+System+Plugin+Timeout+While+Waiting+for+Add-ons+to+Enable)

### Upgrade
    
Refer to [Upgrading JIRA applications manually](https://confluence.atlassian.com/adminjiraserver071/upgrading-jira-applications-manually-802592252.html)

To upgrade to a more recent version of JIRA Software you can simply stop the `jira`
container and start a new one based on a more recent Docker image:

    $> docker stop jira
    $> docker rm jira
    $> docker run ... (See above)

As your data is stored in the data volume directory on the host it will still be available after the upgrade.

_IMPORTANT: Please make sure that you **don't** accidentally remove the `jira`
container and its volumes using the `-v` option._


### Backup 
Please refer to official [Backing up data](https://confluence.atlassian.com/adminjiraserver071/backing-up-data-802592964.html) documentation of JIRA Software before reading further.
TODO - See [this](https://github.com/mhubig/atlassian/tree/master/atlassian-jira) image

### Restore
Please refer to official [Restoring data](https://confluence.atlassian.com/adminjiraserver071/restoring-data-802592977.html) documentation of JIRA Software before reading further.
TODO - See [this](https://github.com/mhubig/atlassian/tree/master/atlassian-jira) image

# License

This image is licensed under the Apache License, Version 2.0. See [LICENSE](LICENSE) for the full license text.

# Supported Docker versions

This image is officially supported on Docker version 1.12.

Support for older versions (down to 1.6) is provided on a best-effort basis.

Please see [the Docker installation documentation](https://docs.docker.com/installation/) for details on how to upgrade your Docker daemon.

# User Feedback

## Documentation

Documentation for this image is currently only in this [README.md](README.md) file. Please support us keeping documentation up to date and relevant.

## Issues

If you have any problems with or questions about this image, please contact us through a [GitHub issue](https://github.com/acntech/docker-jira/issues)

You can also reach image maintainers mentioned in the [Dockerfile](Dockerfile).

## Contributing

You are invited to contribute new features, fixes, or updates, large or small; we are always thrilled to receive pull requests, and do our best to process them as fast as we can.

Before you start to code, we recommend discussing your plans through a [GitHub issue](https://github.com/acntech/docker-jira/issues), especially for more ambitious contributions. This gives other contributors a chance to point you in the right direction, give you feedback on your design, and help you find out if someone else is working on the same thing.
