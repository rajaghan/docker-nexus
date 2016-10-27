# Copyright (c) 2014-present Sonatype, Inc.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#      http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

FROM registry.access.redhat.com/rhel7:latest
MAINTAINER Sonatype <cloud-ops@sonatype.com>
LABEL vendor=Sonatype \
  com.sonatype.license="Apache License, Version 2.0" \
  com.sonatype.name="Nexus Repository Manager OSS base image"

ENV SONATYPE_WORK="/opt/sonatype/sonatype-work" SONATYPE_INSTALL="/opt/sonatype/nexus" NEXUS_VERSION="2.14.1-01"

# ENV JAVA_HOME /opt/java
# ENV JAVA_VERSION_MAJOR 8
# ENV JAVA_VERSION_MINOR 102
# ENV JAVA_VERSION_BUILD 14
             
RUN yum install -y \
  curl tar createrepo java \
  && yum clean all

# install Oracle JRE
#RUN mkdir -p /opt \
#  && curl --fail --silent --location --retry 3 \
#  --header "Cookie: oraclelicense=accept-securebackup-cookie; " \
#  http://download.oracle.com/otn-pub/java/jdk/${JAVA_VERSION_MAJOR}u${JAVA_VERSION_MINOR}-b${JAVA_VERSION_BUILD}/server-jre-${JAVA_VERSION_MAJOR}u${JAVA_VERSION_MINOR}-linux-x64.tar.gz \
#  | gunzip \
#  | tar -x -C /opt \
#  && ln -s /opt/jdk1.${JAVA_VERSION_MAJOR}.0_${JAVA_VERSION_MINOR} ${JAVA_HOME}

RUN mkdir -p ${SONATYPE_INSTALL} \
  && curl --fail --silent --location --retry 3 \
    https://download.sonatype.com/nexus/oss/nexus-${NEXUS_VERSION}-bundle.tar.gz \
  | gunzip \
  | tar x -C /tmp nexus-${NEXUS_VERSION} \
  && mv /tmp/nexus-${NEXUS_VERSION}/* ${SONATYPE_INSTALL} \
  && rm -rf /tmp/nexus-${NEXUS_VERSION}

# RUN useradd -r -u 200 -m -c "nexus role account" -d ${SONATYPE_WORK} -s /bin/false nexus

VOLUME ${SONATYPE_WORK}

EXPOSE 8081
WORKDIR ${SONATYPE_INSTALL}
# USER nexus

# OpenShift permissions
RUN chgrp -R 0 ${SONATYPE_WORK}
RUN chmod -R g+rw ${SONATYPE_WORK}
RUN find ${SONATYPE_WORK} -type d -exec chmod g+x {} +

RUN chgrp -R 0 ${SONATYPE_INSTALL}
RUN chmod -R g+rw ${SONATYPE_INSTALL}
RUN find ${SONATYPE_INSTALL} -type d -exec chmod g+x {} +

ENV MAX_HEAP=768m MIN_HEAP=256m JAVA_OPTS="-server -Djava.net.preferIPv4Stack=true" LAUNCHER_CONF="./conf/jetty.xml ./conf/jetty-requestlog.xml"

CMD java \
  -Dnexus-work=${SONATYPE_WORK} \
  -Dnexus-webapp-context-path=  \
  -Xms${MIN_HEAP} -Xmx${MAX_HEAP} \
  -cp 'conf/:lib/*' \
  ${JAVA_OPTS} \
  org.sonatype.nexus.bootstrap.Launcher ${LAUNCHER_CONF}

