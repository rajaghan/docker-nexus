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
  com.sonatype.name="Nexus Repository Manager Pro base image"

ENV SONATYPE_HOME="/opt/sonatype/" SONATYPE_WORK="/opt/sonatype/sonatype-work" SONATYPE_INSTALL="/opt/sonatype/nexus" NEXUS_VERSION="2.14.1-01"
             
RUN yum install -y \
  curl tar createrepo java \
  && yum clean all

RUN mkdir -p ${SONATYPE_INSTALL} \
  && curl --fail --silent --location --retry 3 \ 
     https://download.sonatype.com/nexus/professional-bundle/nexus-professional-${NEXUS_VERSION}-bundle.tar.gz \
  | gunzip \
  | tar x -C /tmp nexus-professional-${NEXUS_VERSION} \
  && mv /tmp/nexus-professional-${NEXUS_VERSION}/* ${SONATYPE_INSTALL} \
  && rm -rf /tmp/nexus-professional-${NEXUS_VERSION}

EXPOSE 8081
WORKDIR ${SONATYPE_INSTALL}

# OpenShift permissions
RUN chgrp -R 0 ${SONATYPE_HOME}
RUN chmod -R g+rw ${SONATYPE_HOME}
RUN find ${SONATYPE_HOME} -type d -exec chmod g+x {} +

ENV MAX_HEAP=768m MIN_HEAP=256m JAVA_OPTS="-server -Djava.net.preferIPv4Stack=true" LAUNCHER_CONF="./conf/jetty.xml ./conf/jetty-requestlog.xml" CONTEXT_PATH="/nexus"

CMD java \
  -Dnexus-work=${SONATYPE_WORK} \
  -Dnexus-webapp-context-path=  \
  -Xms${MIN_HEAP} -Xmx${MAX_HEAP} \
  -cp 'conf/:lib/*' \
  ${JAVA_OPTS} \
  org.sonatype.nexus.bootstrap.Launcher ${LAUNCHER_CONF}

