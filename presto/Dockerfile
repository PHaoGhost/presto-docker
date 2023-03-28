FROM bellsoft/liberica-openjdk-alpine:17.0.3

LABEL Maintainer="ph-sky"
LABEL Description="Trino(Presto) Server"
LABEL Version="0.2.0"

ENV TRINO_VERSION=410

ENV PRESTO_SERVER_FILE_NAME_WITHOUT_EXTENTION=trino-server-${TRINO_VERSION}

ENV PRESTO_SERVER_FILE_NAME=${PRESTO_SERVER_FILE_NAME_WITHOUT_EXTENTION}.tar.gz

ENV TRINO_CLI_FILE_NAME_WITHOUT_EXTENTION=trino-cli-${TRINO_VERSION}

ENV TRINO_CLI_FILE_NAME=${TRINO_CLI_FILE_NAME_WITHOUT_EXTENTION}-executable.jar

# ENV PRESTO_MD5SUM=d96ad91c6325834b8ed610d09af5493e

# presto server
# https://repo1.maven.org/maven2/io/trino/trino-server/352/trino-server-352.tar.gz
ADD https://repo1.maven.org/maven2/io/trino/trino-server/${TRINO_VERSION}/${PRESTO_SERVER_FILE_NAME} /tmp

# Is an upgrade required?
#RUN apk --no-cache upgrade

# Dependencies: python > 2.4 and uuidgen (in package util-linux)
RUN apk add --no-cache python3 py-pip ca-certificates python3-dev util-linux && pip install --upgrade pip

# Use dig tool to get ip of localstack and use it to configure s3 endpoint url
RUN apk add --update bind-tools

# Need less tool to execute a query for trino-cli
RUN apk add --no-cache less

RUN ln -s /usr/bin/python3 /usr/bin/python
# Only if 'presto' user is required, install:
# /usr/sbin/useradd
# /usr/sbin/groupadd

# Copy default Presto configuration files and simple/default connectors
COPY config/* /usr/lib/presto/etc/
COPY catalog/* /usr/lib/presto/etc/catalog/

# Install Presto Server:
WORKDIR /tmp
RUN echo \
  # Untar presto-server
  && tar xzf ${PRESTO_SERVER_FILE_NAME} \
  # Create 'presto' user and group:
  # Is this really required in a container? If so, replace them with USER because groupadd and useradd are not available in alpine
  # && getent group presto >/dev/null || /usr/sbin/groupadd -r presto \
  # && getent passwd presto >/dev/null || /usr/sbin/useradd --comment "Presto" -s /sbin/nologin -g presto -r -d /var/lib/presto presto \
  # Create destination directories and link to presto configuration
  && install -d -m=755 /var/lib/presto \
  && install -d -m=755 /var/log/presto \
  # Not required, done with the COPY above
  # && mkdir -p /usr/lib/presto/etc \
  && ln -s /usr/lib/presto/etc /etc/presto \
  # Move directories from untar'd file to /usr/lib/presto
  && mv /tmp/${PRESTO_SERVER_FILE_NAME_WITHOUT_EXTENTION}/bin /usr/lib/presto \
  # Make sure launcher scripts are executable
  && find /usr/lib/presto/bin -type f -exec chmod 0755 {} \; \
  && mv /tmp/${PRESTO_SERVER_FILE_NAME_WITHOUT_EXTENTION}/lib /usr/lib/presto \
  && mv /tmp/${PRESTO_SERVER_FILE_NAME_WITHOUT_EXTENTION}/plugin /usr/lib/presto/lib \
  && mkdir -p /usr/shared/doc/presto \
  && mv /tmp/${PRESTO_SERVER_FILE_NAME_WITHOUT_EXTENTION}/README.txt /usr/shared/doc/presto \
  && rm -rf ${PRESTO_SERVER_FILE_NAME_WITHOUT_EXTENTION} ${PRESTO_SERVER_FILE_NAME} \
  # Write the JAVA_HOME to /etc/presto/env.sh
  && echo "JAVA8_HOME=${JAVA_HOME}" > /etc/presto/env.sh \
  # Required only if presto user was created. Make sure those directory exists.
  # && chown -R presto:presto /var/lib/presto \
  # && chown -R presto:presto /var/log/presto \
  # && chown -R presto:presto /etc/presto \
  && echo

COPY etc/init.d/presto /etc/init.d/presto
RUN chmod 0755 /etc/init.d/presto
COPY entrypoint.sh /usr/local/bin/
RUN chmod 0755 /usr/local/bin/entrypoint.sh
COPY startup.sh /usr/local/bin/
RUN chmod 0755 /usr/local/bin/startup.sh

# install presto client
# https://repo1.maven.org/maven2/io/trino/trino-cli/410/trino-cli-410-executable.jar 
ADD https://repo1.maven.org/maven2/io/trino/trino-cli/${TRINO_VERSION}/${TRINO_CLI_FILE_NAME} /usr/local/bin/${TRINO_CLI_FILE_NAME_WITHOUT_EXTENTION}

RUN chmod 0755 /usr/local/bin/${TRINO_CLI_FILE_NAME_WITHOUT_EXTENTION} && \
    ln -s /usr/local/bin/${TRINO_CLI_FILE_NAME_WITHOUT_EXTENTION} /usr/local/bin/presto-cli
    
EXPOSE 8080
WORKDIR /etc/presto

ENTRYPOINT [ "/usr/local/bin/entrypoint.sh" ]
CMD /usr/local/bin/startup.sh
