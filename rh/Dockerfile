FROM docker-nexus:latest
MAINTAINER bholmes@redhat.com

COPY conf/nexus.xml ${SONATYPE_WORK}/conf/nexus.xml

# OpenShift permissions
RUN chgrp -R 0 ${SONATYPE_HOME}
RUN chmod -R g+rw ${SONATYPE_HOME}
RUN find ${SONATYPE_HOME} -type d -exec chmod g+x {} +
