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

    # apt install -y gnupg2 wget bash tini libc6 libpam-modules krb5-user libnss3 procps net-tools gosu \
    #                build-essential gdb lcov pkg-config curl \
    #                libbz2-dev libffi-dev libgdbm-dev libgdbm-compat-dev liblzma-dev \
    #                libncurses5-dev libreadline6-dev libsqlite3-dev libssl-dev \
    #                lzma lzma-dev tk-dev uuid-dev zlib1g-dev dnsutils \
    #                r-base r-base-dev apt-transport-https ca-certificates

# Install Python 3.10.5
# RUN wget https://www.python.org/ftp/python/3.10.5/Python-3.10.5.tgz && \
#     tar -xvf Python-3.10.5.tgz && \
#     cd Python-3.10.5 && \
#     ./configure --enable-optimizations --enable-profiling --with-lto --with-threads && \
#     make -j $(nproc) && \
#     make altinstall && \
#     cd .. && \
#     rm -rf Python-3.10.5 && \
#     rm Python-3.10.5.tgz

# RUN ln -s /usr/local/bin/python3.10 /usr/local/bin/python3 && \
#     ln -s /usr/local/bin/python3.10 /usr/local/bin/python && \
#     ln -s /usr/local/bin/pip3.10 /usr/local/bin/pip3 && \
#     ln -s /usr/local/bin/pip3.10 /usr/local/bin/pip

# RUN pip install --no-cache-dir --upgrade pip && \
#     pip install --no-cache-dir --upgrade setuptools

# Install Java 11
RUN apt install -y ca-certificates curl && \
    curl -s https://repos.azul.com/azul-repo.key | gpg --dearmor -o /usr/share/keyrings/azul.gpg && \
    echo "deb [signed-by=/usr/share/keyrings/azul.gpg] https://repos.azul.com/zulu/deb stable main" | tee /etc/apt/sources.list.d/zulu.list && \
    apt update && \
    apt install -y zulu-11

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
ENV SPARK_TGZ_URL=https://archive.apache.org/dist/spark/spark-3.1.1/spark-3.1.1-bin-hadoop2.7.tgz \
    SPARK_TGZ_ASC_URL=https://archive.apache.org/dist/spark/spark-3.1.1/spark-3.1.1-bin-hadoop2.7.tgz.asc \
    GPG_KEY=3D2F69CEED5E30F6252935903FF7C4E834D9EA44

RUN set -ex; \
    export SPARK_TMP="$(mktemp -d)"; \
    cd $SPARK_TMP; \
    wget -nv -O spark.tgz "$SPARK_TGZ_URL"; \
    wget -nv -O spark.tgz.asc "$SPARK_TGZ_ASC_URL"; \
    export GNUPGHOME="$(mktemp -d)"; \
    gpg --keyserver hkps://keys.openpgp.org --recv-key "$GPG_KEY" | \
    gpg --keyserver hkps://keyserver.ubuntu.com --recv-keys "$GPG_KEY"; \
    gpg --batch --verify spark.tgz.asc spark.tgz; \
    gpgconf --kill all; \
    rm -rf "$GNUPGHOME" spark.tgz.asc; \
    tar -xf spark.tgz --strip-components=1; \
    chown -R spark:spark .; \
    mv jars /opt/spark/; \
    mv bin /opt/spark/; \
    mv sbin /opt/spark/; \
    mv kubernetes/dockerfiles/spark/decom.sh /opt/; \
    mv examples /opt/spark/; \
    mv kubernetes/tests /opt/spark/; \
    mv data /opt/spark/; \
    mv R /opt/spark/; \
    cd ..; \
    rm -rf "$SPARK_TMP";

COPY entrypoint.sh /opt/

ENV SPARK_HOME /opt/spark
ENV R_HOME /usr/lib/R
ENV JAVA_HOME /usr/lib/jvm/zulu11

WORKDIR /opt/spark/work-dir
RUN chmod g+w /opt/spark/work-dir
RUN chmod a+x /opt/decom.sh
RUN chmod a+x /opt/entrypoint.sh

ENTRYPOINT [ "/opt/entrypoint.sh" ]
