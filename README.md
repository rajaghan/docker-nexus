# docker-nexus for openshift

Based on sonatype/docker-nexus - https://github.com/sonatype/docker-nexus

Docker images for Sonatype Nexus Repository Manager 2, using RHEL7 and the Open JDK.

* [Notes](#notes)
  * [Persistent Data](#persistent-data)
  * [Red Hat Nexus Repositories](#red-hat-nexus-repositories)

To build:
```
# oc new-app --strategy=docker --context-dir=oss http://github.com/benemon 
# oc new-app --strategy=docker --context-dir=pro http://github.com/benemon
```
This builds a new instance of Sonatype Nexus in a container.

To expose the Kubernetes service:
```
# oc expose service docker-nexus
```

## Notes

* Context dir is root, unlike a standard Nexus installation, which sits on /nexus

* Default credentials are: `admin` / `admin123`

* It can take some time (2-3 minutes) for the service to launch in a
new container.  You can tail the log to determine once Nexus is ready. In order to do this, you need to identify the pod in which Nexus is running:

```
# oc get pods
```

Once the pod is identified, tail the logs with:
```
$ oc logs -f <pod name>
```

* Installation of Nexus is to `/opt/sonatype/nexus`.  Notably:
  `/opt/sonatype/nexus/conf/nexus.properties` is the properties file.
  Parameters (`nexus-work` and `nexus-webapp-context-path`) defined
  here are overridden in the JVM invocation.

* A persistent directory, `/opt/sonatype/sonatype-work`, is used for configuration,
logs, and storage. This directory needs to be writeable by the Nexus
process, which runs as a randomised UID in OpenShift, belonging to the root group. Therefore, the persistent directory structures are owned by the root group.

* Environment variables can be used to control the JVM arguments

  * `CONTEXT_PATH`, passed as -Dnexus-webapp-context-path.  This is used to define the
  URL which Nexus is accessed.  Defaults to '/nexus'
  * `MAX_HEAP`, passed as -Xmx.  Defaults to `768m`.
  * `MIN_HEAP`, passed as -Xms.  Defaults to `256m`.
  * `JAVA_OPTS`.  Additional options can be passed to the JVM via this variable.
  Default: `-server -XX:MaxPermSize=192m -Djava.net.preferIPv4Stack=true`.
  * `LAUNCHER_CONF`.  A list of configuration files supplied to the
  Nexus bootstrap launcher.  Default: `./conf/jetty.xml ./conf/jetty-requestlog.xml`

  These can be applied to the OpenShift deploymentconfig to control the JVM.

### Persistent Data

Data can be persisted within OpenShift by adding a Persistent Volume Claim to the deploymentconfig. This should be mapped to `/opt/sonatype/sonatype-work`.

### Red Hat Nexus Repositories

Nexus can be configured to proxy and group repositories. The folder `rh` in this repository contains a Dockerfile based on a build of either oss/ or pro/ within the context of the current OpenShift namespace. It assumes that a previous image for `docker-nexus` exists, and will simply overlay a full configuration over the base configuration, which contains all the Red Hat Maven repositories, along with a Group repository call `redhat-all` - an amalgamation of every other Red Hat repository. This is provided for ease of use.

To use, build either oss or pro:
```
# oc new-build --strategy=docker --context-dir=oss http://github.com/benemon 
# oc new-build --strategy=docker --context-dir=pro http://github.com/benemon
```
Note the use of `oc new-build` as opposed to `new-app`, as we don't need this image to be deployed.

Once the build process has completed, you have your base image. From here we can call:
```
# oc new-app --strategy=docker --context-dir=rh http://github.com/benemon --name=nexus
```
Note the use of `--name` in order to differentiate this build from the base image build.

Once the build is finished, you can expose the new Kubernetes service in much the same way as before:
```
# oc expose service nexus
```

