# feffi/nexus-iq-server

A simple Dockerfile running a Sonatype Nexus IQ Server, based on CentOS.

Just expose the neccessary ports 8070 and 8071 to the host.

```
$ docker run -d -p 8070:8070 -p 8071:8071 --name nexus-iq-server feffi/docker-nexus-iq-server
```

To test:

```
$ curl http://localhost:8071/ping
```

To (re)build the image:

Use the Dockerfile from this repository and do the build

```
$ docker build --rm=true --tag=feffi/docker-nexus-iq-server .
```


## Notes

* Default credentials are: `admin` / `admin123`

* All logs are directed to stdout/stderr:

```
$ docker logs -f nexus-iq-server
```
or
```
$ docker logs -f --tail=50 nexus-iq-server
```

* Installation of IQ Server is to `/opt/sonatype/iq-server`.

* A persistent directory, `/sonatype-work`, is used for reports and DB storage files.
  The persistent directory needs to be writable by the Nexus IQ server process, which runs as
  UID 201.

* An environment variable, `JVM_OPTIONS`, used to control the JVM arguments

  ```
  $ docker run -d -p 8070:8070 --name nexus-iq-server -e JVM_OPTIONS="-server -Xmx2g" feffi/docker-nexus-iq-server
  ```


### Persistent Data

There are two general approaches to handling persistent storage requirements
with Docker. See [Managing Data in Containers](https://docs.docker.com/userguide/dockervolumes/)
for additional information.

  1. *Use a data volume container*.  Since data volumes are persistent
  until no containers use them, a container can created specifically for
  this purpose.  This is the recommended approach.

  ```
  $ docker run -d --name nexus-iq-data feffi/docker-nexus-iq-server echo "data container for Nexus IQ server"
  $ docker run -d -p 8070:8070 --name nexus-iq-server --volumes-from nexus-iq-data feffi/docker-nexus-iq-server
  ```

  2. *Mount a host directory as the volume*.  This is not portable, as it
  pins the container to the host and relies on the directory existing with correct permissions on the host.
  However it can be useful in certain situations where this volume needs
  to be assigned to certain specific underlying storage.

  ```
  $ mkdir /some/path/nexus-iq-data && chown -R 201 /some/path/nexus-iq-data
  $ docker run -d -p 8070:8070 --name nexus-iq-server -v /some/path/nexus-iq-data:/sonatype-work feffi/docker-nexus-iq-server
  ```

### Changing IQ Server Configuration

There are two primary ways to update the configuration for nexus-iq-server.

*Pass parameters to the JVM*.  For instance, to change the `baseUrl`:

```
  $ docker run -d -e JVM_OPTIONS="dw.baseUrl=http://someaddress:8060" feffi/docker-nexus-iq-server
```

*Create an image w/ updated `config.yml`*:

* Create a new `config.yml`
* Create a `Dockerfile`:
```
  FROM feffi/nexus-iq-server
  ADD config.yml /opt/sonatype/iq-server/
```
* Create a local image:
```
  $ docker build -t my-nexus-iq-server .
```
* Use this docker image as you normally would: `docker run -d my-nexus-iq-server`

## Running with docker-compose
In case your using docker-compose to setup the nexus-iq-server just use the above listed files.
