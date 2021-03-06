# docker-jira-software: A Docker image for Jira Software.

[![Build status](https://travis-ci.org/ahaasler/docker-jira-software.svg?branch=master)](https://travis-ci.org/ahaasler/docker-jira-software)
[![Docker Repository on Quay.io](https://quay.io/repository/ahaasler/jira-software/status "Docker Repository on Quay.io")](https://quay.io/repository/ahaasler/jira-software)
[![Release](https://img.shields.io/github/release/ahaasler/docker-jira-software.svg?style=flat)](https://github.com/ahaasler/docker-jira-software/releases/latest)
[![Docker Stars](https://img.shields.io/docker/stars/ahaasler/jira-software.svg?style=flat)](https://hub.docker.com/r/ahaasler/jira-software/)
[![Docker Pulls](https://img.shields.io/docker/pulls/ahaasler/jira-software.svg?style=flat)](https://hub.docker.com/r/ahaasler/jira-software/)
[![Image Size](https://images.microbadger.com/badges/image/ahaasler/jira-software.svg)](https://microbadger.com/images/ahaasler/jira-software)

## Features

* Runs on *Oracle Java* 8.
* Ready to be configured with *Nginx* as a reverse proxy (https available).

## Usage

```bash
docker run -d -p 8080:8080 ahaasler/jira-software
```

### Parameters

You can use this parameters to configure your jira instance:

* **-s:** Enables the connector security and sets `https` as connector scheme.
* **-n &lt;proxyName&gt;:** Sets the connector proxy name.
* **-p &lt;proxyPort&gt;:** Sets the connector proxy port.
* **-c &lt;contextPath&gt;:** Sets the context path (do not write the initial /).

This parameters should be given to the entrypoint (passing them after the image):

```bash
docker run -d -p 8080:8080 ahaasler/jira-software <parameters>
```

> If you want to execute another command instead of launching jira you should overwrite the entrypoint with `--entrypoint <command>` (docker run parameter).

### Nginx as reverse proxy

Lets say you have the following *nginx* configuration for jira:

```
server {
	listen                          80;
	server_name                     example.com;
	return                          301 https://$host$request_uri;
}
server {
	listen                          443;
	server_name                     example.com;

	ssl                             on;
	ssl_certificate                 /path/to/certificate.crt;
	ssl_certificate_key             /path/to/key.key;
	location /jira {
		proxy_set_header X-Forwarded-Host $host;
		proxy_set_header X-Forwarded-Server $host;
		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
		proxy_pass http://127.0.0.1:8080;
		proxy_redirect off;
	}
}
```

> This is only an example, please secure you *nginx* better.

For that configuration you should run your jira container with:

```bash
docker run -d -p 8080:8080 ahaasler/jira-software -s -n example.com -p 443 -c jira
```

### Persistent data

The jira home is set to `/data/jira`. If you want to persist your data you should use a data volume for `/data/jira`.

#### Binding a host directory

```bash
docker run -d -p 8080:8080 -v /home/user/jira-data:/data/jira ahaasler/jira-software
```

Make sure that the jira user (with id 547) has read/write/execute permissions.

If security is important follow the Atlassian recommendation:

> Ensure that only the user running Jira can access the Jira home directory, and that this user has read, write and execute permissions, by setting file system permissions appropriately for your operating system.

#### Using a data-only container

1. Create the data-only container and set proper permissions:

	* **Lazy way (preferred)** - Using [docker-jira-data](https://github.com/ahaasler/docker-jira-data "A data-only container for docker-jira"):

		```bash
docker run --name jira-data ahaasler/jira-data
		```

	* *I-wan't-to-know-what-I'm-doing* way:

		```bash
docker run --name jira-data -v /data/jira busybox true
docker run --rm -it --volumes-from jira-data debian bash
		```

		The last command will open a *debian* container. Execute this inside that container:

		```bash
chown 547:root /data/jira; chmod 770 /data/jira; exit;
		```

2. Use it in the jira container:

	```bash
docker run --name jira --volumes-from jira-data -d -p 8080:8080 ahaasler/jira-software
	```

### PostgreSQL external database

A great way to connect your Jira instance with a PostgreSQL database is
using the [docker-jira-postgres](https://github.com/ahaasler/docker-jira-postgres "A PostgreSQL container for docker-jira")
image.

1. Create and name the database container:

	```bash
docker run --name jira-postgres -d ahaasler/jira-postgres
	```

2. Use it in the Jira container:

	```bash
docker run --name jira --link jira-postgres:jira-postgres -d -p 8080:8080 ahaasler/jira-software
	```

3. Connect your Jira instance following the Atlassian documentation:
[Configure your JIRA server to connect to your PostgreSQL database](https://confluence.atlassian.com/display/JIRA/Connecting+JIRA+to+PostgreSQL#ConnectingJIRAtoPostgreSQL-3.ConfigureyourJIRAservertoconnecttoyourPostgreSQLdatabase "Configure your JIRA server to connect to your PostgreSQL database").

>  See [docker-jira-postgres](https://github.com/ahaasler/docker-jira-postgres "A PostgreSQL container for docker-jira")
for more information and configuration options.

## Thanks

* [Docker](https://www.docker.com/ "Docker") for this amazing container engine.
* [PostgreSQL](http://www.postgresql.org/) for this advanced database.
* [Atlassian](https://www.atlassian.com/ "Atlassian") for making great products. Also for their work on [atlassian-docker](https://bitbucket.org/atlassianlabs/atlassian-docker "atlassian-docker repo") which inspired this.
* And specially to you and the entire community.

## License

This image is licensed under the Apache License, Version 2.0. See [LICENSE](LICENSE) for the full license text.
