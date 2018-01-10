Koha Docker Container
===

This project builds a [Docker](https://www.docker.com/) image containg an installation of the library system [Koha](http://koha-community.org/).

More documentation can be found on project [wiki](https://gitlab.deichman.no/digibib/koha-docker/wiki)

Development Quickstart for Deichman
===

This is documentation on the development setup used in Deichman, but should be general enough to be used by anyone.


Clone the two projects
====


# 1. The Koha codebase

https://github.com/Koha-Community/Koha  (upstream) the community code


```
git clone https://gitlab.deichman.no/digibib/koha
```

We keep master in sync with upstream and a local branch for each release with all our local patches on top (eg. release_17.11.01)

```
git checkout release_17.11.01
```

# 2. The Koha Docker project

https://gitlab.deichman.no/digibib/koha (origin)   deichman fork with local patches

This is the project for setting up Koha using docker in the various environments:
* koha_dev   : development and testing of patches
* koha_ci    : automated build used by gitlab runners for creating debian packages and a docker image
* koha_build : using a production-ready docker image

```
git clone https://gitlab.deichman.no/digibib/koha-docker
```

# 3. Run Development

TODO: use Makefile instead
* cd into docker-compose folder in koha-docker project (koha-docker/docker-compose)
* start a docker container using code from git
* mount locally checked out codebase as a volume inside the container

## Build development docker image

KOHAPATH must point to koha-docker root dir

```
source docker-compose.env && KOHAPATH=.. docker-compose -f common.yml -f dev.yml build koha_dev
```

## Run development container

KOHAPATH must point to koha-docker root dir

REPO must point to koha code repo

* this will run container in foreground with STDOUT and will stop and remove if you Ctrl-C
* docker-compose run params explained
* --rm (delete container when stopped)
* --publish=8081:8081 ( publish ports so koha intra pages are accessible outside container )
* --volume=xxx:yyy ( mounting of host code into container hostpath:containerpath )
* --name ( optionally give container a name )
* -d ( detatch )
* --service-ports ( publish all ports from inside container )
* -e (add environment variables e.g. KOHA_CONF=/etc/koha/sites/name/koha-conf.xml)
* --entrypoint ( override entry point , e.g /bin/bash for a simple shell with no services running )

```
source docker-compose.env && \
  KOHAPATH=.. REPO=/home/benjab/github/digibib/koha \
  docker-compose -f common.yml -f dev.yml run --rm --volume=$REPO:/kohadev/kohaclone --publish=8081:8081 koha_dev
```

# 4. Enter container

Container will have an autogenerated name (find it with `docker ps`) if not started with --name

Admin pages are at localhost:8081, with default admin (admin/secret) login

```
docker exec -it dockercompose_koha_dev_run_1 bash
```

# 5. Useful commands inside container

It is a debian jessie distro, koha source code is mounted in /kohadev/kohaclone

A lot of env vars exist used to configure system (use `env` for overview)

* restart service
```
    supervisorctl -u$KOHA_ADMINUSER -p$KOHA_ADMINPASS restart <service> <service>
      plack ( perl multiworker server, need to be restarted for koha to pick up changes in modules, systempreferences etc )
      sip   ( sip server used by self checkout machines, door access, etc )
      ncip_server ( interlibrary loan NCIP service )
      zebra_server ( local Koha search index )
```

* enter mysql

```
koha-mysql name
```

* Koha xml configuration file

    KOHA_CONF=/etc/koha/sites/name/koha-conf.xml

* run tests ( from /kohadev/kohaclone )

```
KOHA_CONF=/etc/koha/sites/name/koha-conf.xml PERL5LIB=. perl t/db_dependent/api/v1/illrequests.t
```