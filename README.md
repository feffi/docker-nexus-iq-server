# feffi/nexus-iq-server

A simple Dockerfile running a Sonatype Nexus IQ Server, based on CentOS.

Just expose the neccessary ports 8070 and 8071 to the host.

```
$ docker run -d -p 8070:8070 -p 8071:8071 --name nexus-iq-server feffi/nexus-iq-server
```

To test:

```
$ curl http://localhost:8071/ping
```

To (re)build the image:

Use the Dockerfile from this repository and do the build

```
$ docker build --rm=true --tag=feffi/nexus-iq-server .
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
  $ docker run -d -p 8070:8070 --name nexus-iq-server -e JVM_OPTIONS="-server -Xmx2g" feffi/nexus-iq-server
  ```


### Persistent Data

There are two general approaches to handling persistent storage requirements
with Docker. See [Managing Data in Containers](https://docs.docker.com/userguide/dockervolumes/)
for additional information.

  1. *Use a data volume container*.  Since data volumes are persistent
  until no containers use them, a container can created specifically for 
  this purpose.  This is the recommended approach.  

  ```
  $ docker run -d --name nexus-iq-data feffi/nexus-iq-server echo "data container for Nexus IQ server"
  $ docker run -d -p 8070:8070 --name nexus-iq-server --volumes-from nexus-iq-data feffi/nexus-iq-server
  ```

  2. *Mount a host directory as the volume*.  This is not portable, as it
  pins the container to the host and relies on the directory existing with correct permissions on the host.
  However it can be useful in certain situations where this volume needs
  to be assigned to certain specific underlying storage.  

  ```
  $ mkdir /some/path/nexus-iq-data && chown -R 201 /some/path/nexus-iq-data
  $ docker run -d -p 8070:8070 --name nexus-iq-server -v /some/path/nexus-iq-data:/sonatype-work feffi/nexus-iq-server
  ```

### Changing IQ Server Configuration

There are two primary ways to update the configuration for nexus-iq-server. 

*Pass parameters to the JVM*.  For instance, to change the `baseUrl`:

```
  $ docker run -d -e JVM_OPTIONS="dw.baseUrl=http://someaddress:8060" feffi/nexus-iq-server
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

# MIT License

Copyright (c) 2017 feffi

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
