FROM --platform=linux/amd64 tomcat:8.5.51-jdk8-openjdk
ENV PYTHONUNBUFFERED 1
ENV TERM xterm

# helpful aliases
COPY /docker/bashrc_to_docker /root/.bashrc

# ARG vars exist only during build
ARG DEBIAN_FRONTEND=noninteractive

# See https://stackoverflow.com/a/55399898
RUN sed -i '/jessie-updates/d' /etc/apt/sources.list  # Now archived

RUN apt-get --allow-releaseinfo-change update -q && \
        apt-get install -y --no-install-recommends \
        maven  ca-certificates-java \
        git-core curl less psmisc rsync \
        software-properties-common \
        build-essential libssl-dev libffi-dev \
        python3 python3-dev python3-virtualenv \
        python3-setuptools python3-pip python3-apt \
        libjpeg-dev swig \
        dbus libdbus-1-dev libdbus-glib-1-dev \
        libsasl2-dev libldap2-dev libssl-dev \
        # for admin convenience - MariaDB client and mysqldump
        default-mysql-client

ARG VIRTUAL_ENV=/home/python3_env
ARG ACTIVATE=${VIRTUAL_ENV}/bin/activate
WORKDIR ${VIRTUAL_ENV}
RUN /usr/bin/python3 -m virtualenv --python=python3 ${VIRTUAL_ENV} && \
        . ${ACTIVATE} && pip3 install --upgrade pip

WORKDIR /home/scraper
COPY ./docker/requirements.txt .
RUN . ${ACTIVATE} && pip3 install -r requirements.txt
COPY ./scraper/scrape.py .
COPY ./scraper/s3-put.py .

RUN rm -rf /var/lib/apt/lists/*

# Maven wants to run in this dir
WORKDIR /home/article_funnel
COPY /demo/pom.xml .
# build all dependencies for offline use
RUN mvn dependency:go-offline -B

# Now copy rest of source
COPY /demo .
RUN mvn clean && \
        mvn --batch-mode install && \
        cp -av target/demo-0.0.1-SNAPSHOT.war /usr/local/tomcat/webapps

WORKDIR /home/data_migration
COPY MySQLArticleDatabase/ .

WORKDIR $CATALINA_HOME
WORKDIR /home/article_funnel
