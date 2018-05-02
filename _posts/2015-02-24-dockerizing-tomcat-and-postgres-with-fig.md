---
published: true
title: Dockerizing Tomcat and Postgres with Fig
---

Let me share some thoughts on how one can use Docker and Fig to run custom webapps in Tomcat and Postgres.

## Tools that we are using
- [Docker](https://www.docker.com) - Gives ability to run _any_ App in a container
- [Fig](http://www.fig.sh) - helps run a combined setup of few Docker containers
- [boot2docker](http://boot2docker.io) - sets up special Virtual Machine and gives ability to run Docker on Mac OS X
- [Tomcat](http://tomcat.apache.org) - Java app server
- [PostgreSQL](http://www.postgresql.org) - SQL Database

## What do we want
We want:
- run a custom set of Java webapps
- custom Tomcat configuration
- simulate development environment
- Webapps are pre-configured to talk with DB on _localhost_
- in Docker

## Installing tools
- On Mac OS X, download the latest [boot2docker distribution]( https://github.com/boot2docker/osx-installer/releases) and install it.
- install Fig with `pip install -U fig`

## Configuring
We will use a custom Tomcat docker, that will pre-install `postgresql-client` and `socat`. The `socat` is used for forwarding a local port 5432 to the remote address. Her's our `Dockerfile`:

```
FROM tomcat:7
RUN apt-get update && apt-get install -y curl postgresql-client socat
```

Next, create the `fig.yml`:

``` yaml
web:
  build: .
  volumes:
   - tomcat:/usr/local/tomcat
   - startTomcatAll.sh:/start.sh
  command: "sh /start.sh"
  links:
   - db
  ports:
   - "8080:8080"
db:
  image: "postgres:9.4"
  ports:
   - "5432:5432"
  volumes:
   - postgres:/data
```

Some info:
- `build` tells Fig to build a custom Docker container, and specifies where the `Dockerfile` is
- `volumes FROM:TO` maps folder from the host OS, into the guest container. In our case, we have a `tomcat` folder which contains the config and the webapps, and a custom start script.
- `command` executes the custom script upon start. _NOTE:_ The last command should output something, or Fig will stop the container. `tail -f catalina.out` will do the job.
- `links` tells what other containers shoul be linked. You will also get the environment variables like `DB_PORT_5432_TCP`, which we will use in our script.
- `ports` tells what ports to map from container into host OS.
- `image` tells what public image from [Docker Hub](https://registry.hub.docker.com) to use.

The `start.sh` script will look like this:

``` bash
#!/bin/sh

# Port forwarding: localhost:5432 -> $DB_PORT_5432_TCP:5432
env | grep DB_PORT_5432_TCP= | sed 's/.*_PORT_\([0-9]*\)_TCP=tcp:\/\/\(.*\):\(.*\)/socat TCP4-LISTEN:\1,fork,reuseaddr TCP4:\2:\3 \&/' | sh

# Check the DB connection by listing tables
psql -h localhost -U postgres -l

# Some scripts rely on the operating system...
echo "Ubuntu" > /etc/issue

# Start the custom Tomcat
startTomcat.sh

# Output the log, so Fig can catch this
tail -f /usr/local/tomcat/bin/catalina.out
```

## A side note about Postgres container
A PostgreSQL official Docker container is set to be run from a user `postgres`. In Mac OS X the `volumes` are mounted with a current Mac user.

If we will mount a volume that contains the binary database files, when a container starts, Postgres will crash, because the username that is used to run a postgres does not match the username that the database files are.

The possible way to overcome is to `tar` the files and "untar" them on start of a container. 

The other way is to import the SQL dump - `psql -f /data/all_dbs.sql -h localhost -U postgre`.

If you are on the Mac, you might want to tell `boot2docker` VM to forward the `5432` port to your Mac. to do so, run 

```
VBoxManage modifyvm "boot2docker-vm" --natpf1 "tcp-port5432,tcp,,5432,,5432";
```

## Starting up

If you'r on a Mac, make sure the boot2docker VM is started - `boot2docker start`. Next, make sure that the environment is properly set by running `$(boot2docker shellinit)`.

Run `fig up` and watch the Fig will start the `db` and `web` containers and outputs the log into the console. You could run `docker build .` to build your `Dockerfile`, but fig will do it for you. 

Now you can open your browser on `localhost:8080` and see your Tomcat app.

You can connect to any container from other shell by running `docker exec -i -t test_web_1 bash` (or `_db_1`)

## Cleanup
- `docker images` will list all images (`-a` will list all)
- `docker ps` will list running containers (`-a` for all)
- `fig rm` will remove all stopped containers
- `docker images -f dangling=true -q |xargs docker rmi -f` - will remove all dangling (not-named) Docker images