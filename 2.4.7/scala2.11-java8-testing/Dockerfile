FROM debian:testing-slim

ARG spark_uid=185

RUN groupadd --system --gid=${spark_uid} spark && \
    useradd --system --uid=${spark_uid} --gid=spark spark

RUN set -ex && \
    apt-get update && \
    ln -s /lib /lib64 && \
    apt upgrade -y && \
    apt install -y gnupg2 wget bash tini libc6 libpam-modules krb5-user libnss3 procps net-tools gosu \
                   r-base r-base-dev

# Install Java 11
RUN apt install -y ca-certificates curl && \
    curl -s https://repos.azul.com/azul-repo.key | gpg --dearmor -o /usr/share/keyrings/azul.gpg && \
    echo "deb [signed-by=/usr/share/keyrings/azul.gpg] https://repos.azul.com/zulu/deb stable main" | tee /etc/apt/sources.list.d/zulu.list && \
    apt update && \
    apt install -y zulu-8

#  clean up
RUN apt-get clean && \
    rm -rf /var/cache/apt/* && \
    rm -rf /var/lib/apt/lists/*

# Install Spark
RUN mkdir -p /opt/spark && \
    mkdir -p /opt/spark/examples && \
    mkdir -p /opt/spark/work-dir && \
    touch /opt/spark/RELEASE && \
    chown -R spark:spark /opt/spark && \
    rm /bin/sh && \
    ln -sv /bin/bash /bin/sh && \
    echo "auth required pam_wheel.so use_uid" >> /etc/pam.d/su && \
    chgrp root /etc/passwd && chmod ug+rw /etc/passwd

# Install Apache Spark
# https://downloads.apache.org/spark/KEYS
ENV SPARK_TGZ_URL=https://archive.apache.org/dist/spark/spark-2.4.7/spark-2.4.7-bin-without-hadoop.tgz \
    SPARK_TGZ_ASC_URL=https://archive.apache.org/dist/spark/spark-2.4.7/spark-2.4.7-bin-without-hadoop.tgz.asc \
    GPG_KEY=66827234BC7B750E

RUN set -ex; \
    export SPARK_TMP="$(mktemp -d)"; \
    cd $SPARK_TMP && \
    wget -nv -O spark.tgz "$SPARK_TGZ_URL" && \
    wget -nv -O spark.tgz.asc "$SPARK_TGZ_ASC_URL" && \
    export GNUPGHOME="$(mktemp -d)" && \
    gpg --keyserver hkps://keys.openpgp.org --recv-key "$GPG_KEY" | \
    gpg --keyserver hkps://keyserver.ubuntu.com --recv-keys "$GPG_KEY" && \
    gpg --batch --verify spark.tgz.asc spark.tgz && \
    gpgconf --kill all && \
    rm -rf "$GNUPGHOME" spark.tgz.asc && \
    tar -xf spark.tgz --strip-components=1 && \
    chown -R spark:spark . && \
    mv jars /opt/spark/ && \
    mv bin /opt/spark/ && \
    mv sbin /opt/spark/ && \
    mv examples /opt/spark/ && \
    mv kubernetes/tests /opt/spark/ && \
    mv data /opt/spark/ && \
    mv R /opt/spark/ && \
    mv conf /opt/spark/ && \
    mv kubernetes/dockerfiles/spark/entrypoint.sh /opt/ && \
    cd .. && \
    rm -rf "$SPARK_TMP"

# install hadoop
ARG HADOOP_VERSION=2.7.7
RUN set -ex && \
    export HADOOP_TMP="$(mktemp -d)" && \
    cd $HADOOP_TMP && \
    wget -q https://archive.apache.org/dist/hadoop/common/hadoop-${HADOOP_VERSION}/hadoop-${HADOOP_VERSION}.tar.gz && \
    tar -xzf hadoop-${HADOOP_VERSION}.tar.gz && \
    mv hadoop-${HADOOP_VERSION} /opt/hadoop && \
    cd .. && \
    rm -rf "$HADOOP_TMP"

# install hive
ARG HIVE_VERSION=1.2.1
RUN set -ex && \
    export HIVE_TMP="$(mktemp -d)" && \
    cd $HIVE_TMP && \
    wget -q https://archive.apache.org/dist/hive/hive-${HIVE_VERSION}/apache-hive-${HIVE_VERSION}-bin.tar.gz && \
    tar -xzf apache-hive-${HIVE_VERSION}-bin.tar.gz && \
    mv apache-hive-${HIVE_VERSION}-bin /opt/hive && \
    rm apache-hive-${HIVE_VERSION}-bin.tar.gz && \
    cd .. && \
    rm -rf "$HIVE_TMP"

# install jline
ARG JLINE_VERSION=2.14.6
RUN set -ex && \
    wget -q -O /opt/spark/jars/jline-${JLINE_VERSION}.jar \
    https://repo1.maven.org/maven2/jline/jline/${JLINE_VERSION}/jline-${JLINE_VERSION}.jar

# install spark-hive
ARG SPARK_VERSION=2.4.7
ARG SCALA_VERSION=2.11
RUN set -ex && \
    wget -q -O /opt/spark/jars/spark-hive_${SCALA_VERSION}-${SPARK_VERSION}.jar \
    https://repo1.maven.org/maven2/org/apache/spark/spark-hive_${SCALA_VERSION}/${SPARK_VERSION}/spark-hive_${SCALA_VERSION}-${SPARK_VERSION}.jar

# install hive jdbc
# https://repo1.maven.org/maven2/org/apache/hive/hive-jdbc/1.2.1/hive-jdbc-1.2.1-standalone.jar
RUN set -ex && \
    wget -q -O /opt/spark/jars/hive-jdbc-${HIVE_VERSION}-standalone.jar \
    https://repo1.maven.org/maven2/org/apache/hive/hive-jdbc/${HIVE_VERSION}/hive-jdbc-${HIVE_VERSION}-standalone.jar

# COPY entrypoint.sh /opt/
COPY decom.sh /opt/

ENV SPARK_HOME /opt/spark
ENV R_HOME /usr/lib/R
ENV JAVA_HOME /usr/lib/jvm/zulu8
ENV HADOOP_HOME=/opt/hadoop
ENV HIVE_HOME=/opt/hive
ENV PATH=$PATH:$JAVA_HOME/bin:$SPARK_HOME/bin:$HADOOP_HOME/bin:$HIVE_HOME/bin

RUN cp $SPARK_HOME/conf/spark-env.sh.template $SPARK_HOME/conf/spark-env.sh && \
    echo "export HADOOP_CLASSPATH=\$(/opt/hadoop/bin/hadoop classpath)" >> $SPARK_HOME/conf/spark-env.sh && \
    echo "export HIVE_CLASSPATH=\$(find "/opt/hive/lib" -name '*.jar' | xargs echo | tr ' ' ':')" >> $SPARK_HOME/conf/spark-env.sh && \
    echo "export SPARK_DIST_CLASSPATH=\$HADOOP_CLASSPATH:\$HIVE_CLASSPATH" >> $SPARK_HOME/conf/spark-env.sh

WORKDIR /opt/spark/work-dir
RUN chmod g+w /opt/spark/work-dir
RUN chmod a+x /opt/decom.sh
RUN chmod a+x /opt/entrypoint.sh

ENTRYPOINT [ "/opt/entrypoint.sh" ]
