## Running PMM Server via Docker

*Docker* images of *PMM Server* are stored at the [https://hub.docker.com/r/percona/pmm-server/tags/](percona/pmm-server) public
repository. The host must be able to run *Docker* 1.12.6 or later, and have
network access.

*PMM* needs roughly 1GB of storage for each monitored database node with data
retention set to one week. Minimum memory is 2 GB for one monitored database
node, but it is not linear when you add more nodes.  For example, data from 20
nodes should be easily handled with 16 GB.

Make sure that the firewall and routing rules of the host do not constrain the
*Docker* container. For more information, see [https://www.percona.com/doc/percona-monitoring-and-management/faq.html#troubleshoot-connection](How to troubleshoot communication issues between PMM Client and PMM Server).

For more information about using *Docker*, see the [https://docs.docker.com/](Docker Docs).

*By default, [https://www.percona.com/doc/percona-monitoring-and-management/faq.html#data-retention](retention) is set to 30 days for Metrics Monitor and to 8 days for PMM Query Analytics.  Also consider [https://www.percona.com/doc/percona-monitoring-and-management/faq.html#performance-issues](disabling table statistics)? which can greatly decrease Prometheus database size.*

<details>
  <summary><h3>Setting Up a Docker Container for PMM Server</h3></summary>

A Docker image is a collection of preinstalled software which enables running
a selected version of PMM Server on your computer. A Docker image is not run
directly. You use it to create a Docker container for your PMM Server. When
launched, the Docker container gives access to the whole functionality of
PMM.

The setup begins with pulling the required Docker image. Then, you proceed by
creating a special container for persistent PMM data. The last step is
creating and launching the PMM Server container.

### Pulling the PMM Server Docker Image

To pull the latest version from Docker Hub:

```bash
   $ docker pull percona/pmm-server:latest
```

This step is not required if you are running PMM Server for the first time.
However, it ensures that if there is an older version of the image tagged with
`latest` available locally, it will be replaced by the actual latest
version.

### Creating the pmm-data Container

To create a container for persistent PMM data, run the following command:

```bash
   $ docker create \
      -v /opt/prometheus/data \
      -v /opt/consul-data \
      -v /var/lib/mysql \
      -v /var/lib/grafana \
      --name pmm-data \
      percona/pmm-server:latest /bin/true
```

**Note:** *This container does not run, it simply exists to make sure you retain  all PMM data when you upgrade to a newer PMM Server image.  Do not remove or re-create this container, unless you intend to wipe out all PMM data and start over.*

The previous command does the following:

* The **docker create** command instructs the Docker daemon
  to create a container from an image.

* The `-v` options initialize data volumes for the container.

* The `--name` option assigns a custom name for the container
  that you can use to reference the container within a Docker network.
  In this case: `pmm-data`.

* `percona/pmm-server:latest` is the name and version tag of the image
  to derive the container from.

* `/bin/true` is the command that the container runs.

**Important:** *Make sure that the data volumes that you initialize with the `-v` option match those given in the example. PMM Server expects that those directories are bind mounted exactly as demonstrated.*

### Creating and Launching the PMM Server Container

To create and launch PMM Server in one command, use **docker run**:

```bash
   $ docker run -d \
      -p 80:80 \
      --volumes-from pmm-data \
      --name pmm-server \
      --restart always \
      percona/pmm-server:latest
```

This command does the following:

* The **docker run** command runs a new container based on the
  |opt.pmm-server.latest| image.

* The `-d` option starts the container in the background (detached mode).

* The `-p` option maps the port for accessing the PMM Server web UI.
  For example, if port **80** is not available,
  you can map the landing page to port 8080 using ``-p 8080:80``.

* The `-v` option mounts volumes
  from the `pmm-data` container (see [https://www.percona.com/doc/percona-monitoring-and-management/deploy/server/docker.setting-up.html#data-container](Creating the pmm-data Container)).

* The `--name` option assigns a custom name to the container
  that you can use to reference the container within the Docker network.
  In this case: ``pmm-server``.

* The `--restart` option defines the container's restart policy.
  Setting it to ``always`` ensures that the Docker daemon
  will start the container on startup
  and restart it if the container exits.

* `percona/pmm-server:latest` is the name and version tag of the image
  to derive the container from.

### Installing and using specific docker version

To install specific PMM Server version instead of the latest one, just put
desired version number after the colon. Also in this scenario it may be useful
to [https://www.percona.com/doc/percona-monitoring-and-management/glossary.option.html](prevent updating PMM Server via the web interface) with the `DISABLE_UPDATES` docker option.

For example, installing version 1.14.1 with disabled update button in the web
interface would look as follows:

```bash
   $ docker create \
      -v /opt/prometheus/data \
      -v /opt/consul-data \
      -v /var/lib/mysql \
      -v /var/lib/grafana \
      --name pmm-data \
      percona/pmm-server:1.14.1 /bin/true

   $ docker run -d \
      -p 80:80 \
      --volumes-from pmm-data \
      --name pmm-server \
      -e DISABLE_UPDATES=true \
      --restart always \
      percona/pmm-server:1.14.1
```

### Additional options

When running the PMM Server, you may pass additional parameters to the
**docker run** subcommand. All options that appear after the `-e` option
are the additional parameters that modify the way how PMM Server operates.

The section [https://www.percona.com/doc/percona-monitoring-and-management/glossary.option.html#pmm-glossary-pmm-server-additional-option](PMM Server Additional Options) lists all
supported additional options.

</details>

<details>
  <summary><h2>Updating PMM Server Using Docker</h2></summary>

To check the version of PMM Server, run **docker ps** on the host.

Run the following commands as root or by using the **sudo** command

```bash
   $ docker ps
   CONTAINER ID   IMAGE                      COMMAND                CREATED       STATUS             PORTS                               NAMES
   480696cd4187   percona/pmm-server:1.4.0   "/opt/entrypoint.sh"   4 weeks ago   Up About an hour   192.168.100.1:80->80/tcp, 443/tcp   pmm-server
```

The version number is visible in the Image column. For a Docker
container created from the image tagged `latest`, the Image column
contains `latest` and not the specific version number of PMM Server.

The information about the currently installed version of PMM Server is
available from the |srv.update.main.yml| file. You may extract the version
number by using the **docker exec** command:

```bash
   $ docker exec -it pmm-server head -1 /srv/update/main.yml
   # v1.5.3
```

To check if there exists a newer version of PMM Server,
visit [https://hub.docker.com/r/percona/pmm-server/tags/](percona/pmm-server).

.. _pmm.deploying.server.docker-container.renaming:

### Creating a backup version of the current pmm-server Docker container

You need to create a backup version of the current `pmm-server` container if
the update procedure does not complete successfully or if you decide not to
upgrade your PMM Server after trying the new version.

The **docker stop** command stops the currently running `pmm-server` container:

```bash
   $ docker stop pmm-server
```

The following command simply renames the current `pmm-server` container to
avoid name conflicts during the update procedure:

```bash
   $ docker rename pmm-server pmm-server-backup
```

### Pulling a new Docker Image

Docker images for all versions of PMM are available from
[https://hub.docker.com/r/percona/pmm-server/tags/](percona/pmm-server)
Docker repository.

When pulling a newer Docker image, you may either use a specific version
number or the `latest` image which always matches the highest version
number. 

This example shows how to pull a specific version:

```bash
   $ docker pull percona/pmm-server:1.5.0
```

This example shows how to pull the `--latest` version:
   
```bash
   $ docker pull percona/pmm-server:latest
```
   
### Creating a new Docker container based on the new image

After you have pulled a new version of PMM from the Docker repository, you can
use **docker run** to create a `pmm-server` container using the new image.

```bash
   $ docker run -d \
      -p 80:80 \
      --volumes-from pmm-data \
      --name pmm-server \
      --restart always \
      percona/pmm-server:latest
```

**Important:** *The PMM Server container must be stopped before attempting `docker run`.*

The **docker run** command refers to the pulled image as the last parameter. If
you used a specific version number when running **docker pull** (see
[https://www.percona.com/doc/percona-monitoring-and-management/deploy/server/docker.setting-up.html#pmm-server-docker-image-pulling](Pulling the PMM Server Docker Image)) replace `latest` accordingly.

Note that this command also refers to `pmm-data` as the value of
`--volumes-from` option. This way, your new version will continue to use the
existing data.

**Warning:** *Do not remove the `pmm-data` container when updating, if you want to keep all collected data.*

Check if the new container is running using **docker ps**.

```bash
   $ docker ps
   CONTAINER ID   IMAGE                      COMMAND                CREATED         STATUS         PORTS                               NAMES
   480696cd4187   percona/pmm-server:1.5.0   "/opt/entrypoint.sh"   4 minutes ago   Up 4 minutes   192.168.100.1:80->80/tcp, 443/tcp   pmm-server

```

Then, make sure that the PMM version has been updated (see [https://www.percona.com/doc/percona-monitoring-and-management/glossary.terminology.html#term-pmm-version](PMM
Version)) by checking the PMM Server web interface.

### Removing the backup container

After you have tried the features of the new version, you may decide to
continupe using it. The backup container that you have stored
([https://www.percona.com/doc/percona-monitoring-and-management/deploy/server/docker.upgrading.html#pmm-deploying-server-docker-container-renaming](Creating a backup version of the current pmm-server Docker container)) is no longer needed in this
case.

To remove this backup container, you need the **docker rm** command:

```bash
   $ docker rm pmm-server-backup
```

As the parameter to **docker rm**, supply the tag name of your backup container.

If, for whatever reason, you decide to keep using the old version, you just need
to stop and remove the new `pmm-server` container.

```bash
   $ docker stop pmm-server && docker rm pmm-server
```

Now, rename the `pmm-server-backup` to `pmm-server`
(see [https://www.percona.com/doc/percona-monitoring-and-management/deploy/server/docker.upgrading.html#pmm-deploying-server-docker-container-renaming](Creating a backup version of the current pmm-server Docker container)) and start it.

```bash
   $ docker start pmm-server
```

**Warning:** *Do not use the `docker run` command to start the container. The `docker.run` command creates and then runs a new container. To start a new container use the `docker start` command.*

</details>


