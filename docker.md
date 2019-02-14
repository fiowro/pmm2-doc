# Running PMM Server via Docker

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

<details><summary>Setting Up a Docker Container for PMM Server</summary>
<p>

A Docker image is a collection of preinstalled software which enables running
a selected version of PMM Server on your computer. A Docker image is not run
directly. You use it to create a Docker container for your PMM Server. When
launched, the Docker container gives access to the whole functionality of
PMM.

The setup begins with pulling the required Docker image. Then, you proceed by
creating a special container for persistent PMM data. The last step is
creating and launching the PMM Server container.

## Pulling the PMM Server Docker Image

To pull the latest version from Docker Hub:

```bash
   $ docker pull percona/pmm-server:latest
```

This step is not required if you are running PMM Server for the first time.
However, it ensures that if there is an older version of the image tagged with
`latest` available locally, it will be replaced by the actual latest
version.

## Creating the pmm-data Container

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

## Creating and Launching the PMM Server Container

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

## Installing and using specific docker version

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


## Additional options

When running the PMM Server, you may pass additional parameters to the
**docker run** subcommand. All options that appear after the `-e` option
are the additional parameters that modify the way how PMM Server operates.

The section [https://www.percona.com/doc/percona-monitoring-and-management/glossary.option.html#pmm-glossary-pmm-server-additional-option](PMM Server Additional Options) lists all
supported additional options.

</p>
</details>
