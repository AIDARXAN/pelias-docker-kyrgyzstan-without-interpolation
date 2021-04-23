# Pelias in Docker

This repository contains a framework for downloading/preparing and building the [Pelias Geocoder](https://github.com/pelias/pelias) using Docker and [Docker Compose](https://github.com/docker/compose#docker-compose).

## Prerequisites

You will need to have a [modern version of `docker`](https://docs.docker.com/engine/release-notes/) and a [modern version of `docker-compose`](https://github.com/docker/compose/blob/master/CHANGELOG.md) installed before continuing. If you are not using the latest version, please mention that in any bugs reports.

This project supports Linux and Mac OSX operatings systems. Windows is currently [not supported](https://github.com/pelias/docker/issues/124).

### Permissions

In order to ensure security, Pelias docker containers, and the `pelias` helper script, will not run as a root user!

Be sure you are running as a non-root user and that this user can execute `docker` commands. See the Docker documentation article [Manage Docker as a non-root user](https://docs.docker.com/engine/install/linux-postinstall/#manage-docker-as-a-non-root-user) to do this.

## Requirements for Linux
- Install `util-linux` using your distribution's package manager
  - Alpine Linux: `sudo apk add util-linux`
  - Debian/Ubuntu: `sudo apt-get install util-linux` 

## Requirements for Mac OSX
- install GNU coreutils with [Homebrew](https://brew.sh/): `brew install coreutils`.
- Max-out Docker computing resources( `Memory-RAM and CPUs-Cores` ) dedicated to Docker in `Docker > Preferences > Advanced`.

## System requirements

Scripts can easily download tens of GB of geographic data, so ensure you have enough free disk space!

At least 6GB RAM is required.

## Quickstart build script

The following shell script can be used to quickly get started with a Pelias build.

```bash
#!/bin/bash
set -x

# change directory to the where you would like to install Pelias
# cd /path/to/install

# clone this repository
git clone https://github.com/AIDARXAN/pelias-docker-kyrgyzstan-without-interpolation.git && cd docker

# install pelias script
# this is the _only_ setup command that should require `sudo`
sudo ln -s "$(pwd)/pelias" /usr/local/bin/pelias

# cd into the project directory
cd projects/kyrgyzstan

# create a directory to store Pelias data files
# see: https://github.com/pelias/docker#variable-data_dir
# note: use 'gsed' instead of 'sed' on a Mac
mkdir ./data
sed -i '/DATA_DIR/d' .env
echo 'DATA_DIR=./data' >> .env

# run build
pelias compose pull
pelias elastic start
pelias elastic wait
pelias elastic create
pelias download all
pelias prepare all
pelias import all
pelias compose up

# optionally run tests
pelias test run
```


### Resolving PATH issues

If you are having trouble getting this to work then quickly check that the target of the symlink is listed on your $PATH:

```bash
tr ':' '\n' <<< "$PATH"
```

If you used the `ln -s` command above then the directory `/usr/local/bin` should be listed in the output.

If the symlink target path is *not* listed, then you will either need to add its location to your $PATH or create a new symlink which points to a location which is already on your $PATH.




## Optionally cleanup temporary files

Once the build is complete, you can cleanup temporary files that are no longer useful. The numbers in this snippet below are rough estimates for a full planet build.

```
# These folders can be entirely deleted after the import into elastic search
rm -rf /data/openaddresses #(~43GB)
rm -rf /data/tiger #(~13GB)
rm -rf /data/openstreetmap #(~46GB)
rm -rf /data/polylines #(~2.7GB)

# Within the content of the "interpolation" folder (~176GB) we must
# preserve "street.db" (~7GB) and "address.db" (~25GB), the rest can be deleted
cd /data/interpolation
rm -rf -- !("street.db"|"address.db")

# Within the content of the "placeholder" folder (~1.4GB), we must
# preserve the "store.sqlite3" (~0.9GB) file, the rest can be deleted
cd /data/placeholder
rm -rf -- !("store.sqlite3")
```

## View status of running containers

Once the build is complete, you can view the current status and port mappings of the Pelias docker containers:

```bash
pelias compose ps
```

## View logs and debug errors

You can inspect the container logs for errors by running:

```bash
pelias compose logs
```

## Example queries

Once all the importers have completed and the Pelias services are running, you can make queries against your new Pelias build:

### API

- http://localhost:4000/v1/search?text=portland
- [http://localhost:4000/v1/search?text=1901 Main St](http://localhost:4000/v1/search?text=1901%20Main%20St)
- http://localhost:4000/v1/reverse?point.lon=-122.650095&point.lat=45.533467

### Placeholder

- http://localhost:4100/demo/#eng

### PIP (point in polygon)

- http://localhost:4200/-122.650095/45.533467

### Interpolation

- http://localhost:4300/demo/#13/45.5465/-122.6351
