# Java STI builder image

This is a STI builder image for Java builds whose result can be run
directly without any further application server. It's suited ideally
for microservices. The application can be either provided as an
executable FAT Jar or in the more classical with JARs on a flat
classpath with one Main-Class.

This image also provides an easy integration with [agent-bond][1]
which wraps an [Jolokia][2] and Prometheus [jmx_exporter][3]
agent. See below how to configure this.

The following environment variables can be used to influence the
behaviour of this builder image:

## Build-Time

* **STI_DIR** Base STI dir (default: `/tmp`)
* **STI_SOURCE_DIR** Directory where the source should be copied to (default: `${STI_DIR}/src`)
* **STI_ARTIFACTS_DIR** Artifact directory used for incremental build (default: `${STI_DIR}/artifacts`)
* **OUTPUT_DIR** Directory where to find the generated artefacts (default: `${STI_SOURCE_DIR}/target`)
* **JAVA_APP_DIR** Where the application jar should be put to (default: `/app`)

## Run-Time

The run script can be influenced by the following environment variables:

* **JAVA_APP_DIR** the directory where all JAR files can be
  found. This is `/app` by default.
* **JAVA_WORKDIR** working directory from where to start the JVM. By
  default it is `$JAVA_APP_DIR`
* **JAVA_OPTIONS** options to add when calling `java`
* **JAVA_MAIN_CLASS** A main class to use as argument for `java`. When
  this environment variable is given, all jar files in `$JAVA_APP_DIR`
  are added to the classpath as well as `$JAVA_APP_DIR` and
  `JAVA_WORKDIR` themselves, too.
* **JAVA_APP_JAR** A jar file with an appropriate mainfest so that it
  can be started with `java -jar`. If given it takes precedence of
  `$JAVA_MAIN_CLASS`. In addition `$JAVA_APP_DIR` and `$JAVA_WORKDIR`
  are added to the classpath, too. 

If neither `$JAVA_APP_JAR` nor `$JAVA_MAIN_CLASS` is given,
`$JAVA_APP_DIR` is checked for a single JAR file which is taken as
`$JAVA_APP_JAR`. If no or more then one jar file is found, the script
throws an error. 

Any arguments given during startup are taken over as arguments to the
Java call. E.g. a `docker run myimage-based-on-java arg1 arg2` will
be hand over the arguments to the Java application.


The environment variables are best set in `.sti/environment` top in
you project. This file is picked up bei STI during building and running.  

## Agent-Bond Options

Agent bond itself can be influenced via environment variables, too: 

* **AB_OFF** : If set disables activation of agent-bond (i.e. echos an empty value). By default, agent-bond is enabled.
* **AB_ENABLED** : Comma separated list of sub-agents enabled. Currently allowed values are `jolokia` and `jmx_exporter`. 
  By default both are enabled.


#### Jolokia configuration

* **AB_JOLOKIA_CONFIG** : If set uses this file (including path) as Jolokia JVM agent properties (as described 
  in Jolokia's [reference manual](http://www.jolokia.org/reference/html/agents.html#agents-jvm)). 
  By default this is `/opt/jolokia/jolokia.properties`. If this file exists, it be will taken 
  as configuration and **any other config options are ignored**.  
* **AB_JOLOKIA_HOST** : Host address to bind to (Default: `0.0.0.0`)
* **AB_JOLOKIA_PORT** : Port to use (Default: `8778`)
* **AB_JOLOKIA_USER** : User for authentication. By default authentication is switched off.
* **AB_JOLOKIA_PASSWORD** : Password for authentication. By default authentication is switched off.
* **AB_JOLOKIA_ID** : Agent ID to use (`$HOSTNAME` by default, which is the container id)
* **AB_JOLOKIA_OPTS**  : Additional options to be appended to the agent opts. They should be given in the format 
  "key=value,key=value,..."

Some options for integration in various environments

* **AB_JOLOKIA_AUTH_OPENSHIFT** : Switch on OAuth2 authentication for OpenShift. The value of this parameter must be the OpenShift API's 
  base URL (e.g. `https://localhost:8443/osapi/v1/`)

#### jmx_exporter configuration 

* **AB_JMX_EXPORTER_OPTS** : Configuration to use for `jmx_exporter` (in the format `<port>:<path to config>`)
* **AB_JMX_EXPORTER_PORT** : Port to use for the JMX Exporter. Default: `9779`
* **AB_JMX_EXPORTER_CONFIG** : Path to configuration to use for `jmx_exporter`: Default: `/opt/agent-bond/jmx_exporter_config.json`



[1]: https://github.com/fabric8io/agent-bond
[2]: https://github.com/rhuss/jolokia
[3]: https://github.com/prometheus/jmx_exporter