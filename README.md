## Docker orchestration for Data Repositories portals

Docker orchestration for Data Repositories portals


### Installation

1. Install [Docker](https://www.docker.com/).

2. Install [Docker Compose](https://docs.docker.com/compose/).


### Usage for CDR staging

    $ git clone https://github.com/olimpiurob/eea.docker.reportek.git
    $ cd eea.docker.reportek


We need to create the apache-staging.env and cron-staging.env files based
on the sample files provided in the envs directory filling the placeholders with the required info.


    $ docker-compose -f docker-compose-staging.yml up -d --no-recreate
    $ docker-compose -f docker-compose-staging.yml logs


### Upgrade CDR staging

    $ docker-compose -f docker-compose-staging.yml stop
    $ docker-compose -f docker-compose-staging.yml pull
    $ docker-compose -f docker-compose-staging.yml rm -v apache instance1 instance2 zeoserver localconv cron redis haproxy
    $ docker-compose -f docker-compose-staging.yml up -d --no-recreate
    $ docker-compose -f docker-compose-staging.yml logs


## Persistent data as you wish

For production use, in order to avoid data loss we advise you to keep your `Data.fs` and `blobs` within
a [data-only container](https://medium.com/@ramangupta/why-docker-data-containers-are-good-589b3c6c749e).
The `data` container keeps the persistent data for a production environment and must be backed up.
If you are running in a devel environment, you can skip the backup and delete the container if you want.

If you have a `Data.fs` file for your application, you can add it to the `data` container with the following
command:

    $ docker run --rm \
      --volumes-from my_data_container \
      --volume /host/path/to/Data.fs:/restore/Data.fs:ro \
      busybox \
        sh -c "cp /restore/Data.fs /opt/zeoserver/var/filestorage && \
        chown -R 500:500 /opt/zeoserver/var/filestorage"

The command above creates a bare `busybox` container using the persistent volumes of your data container.
The parent directory of the `Data.fs` file is mounted as a `read-only` volume in `/restore`, from where the
`Data.fs` file is copied to the filestorage directory you are going to use (default `/opt/zeoserver/var/filestorage`).
The `data` container must have this directory marked as a volume, so it can be used by the `zope` container,
with a command like:

The data container can also be easily [copied, moved and be reused between different environments](https://docs.docker.com/userguide/dockervolumes/#backup-restore-or-migrate-data-volumes).
