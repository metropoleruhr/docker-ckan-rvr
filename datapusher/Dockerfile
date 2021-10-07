FROM debian:10
LABEL maintainer="David Arndt <arndt@rvr.ruhr>"
# Set timezone
ENV TZ='Europe/Berlin'
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
# Setting the locale   
RUN apt-get update
RUN DEBIAN_FRONTEND=noninteractive apt-get install --no-install-recommends -y locales
RUN DEBIAN_FRONTEND=noninteractive locale-gen de_DE.UTF-8
#Upgrade all installed Packages
RUN apt-get update && apt-get upgrade -y
# Install required system packages
RUN apt-get -q -y update \
    && DEBIAN_FRONTEND=noninteractive apt-get -q -y upgrade \
    && apt-get -q -y install \
    python2 \
    python2-dev \
    python-pip \
    virtualenv \
    python-wheel \
    libpq-dev \
    libxml2-dev \
    libxslt-dev \
    libgeos-dev \
    libssl-dev \
    libffi-dev \
    libsasl2-dev  \
    libldap2-dev \
    libssl-dev \
    postgresql-client \
    build-essential \
    git-core \
    wget \
    curl \
    apache2 \
    apache2-utils \
    libapache2-mod-wsgi \
    zlib1g-dev \
    && apt-get -q clean \
    && rm -rf /var/lib/apt/lists/*
# Define environment variables
RUN mkdir /usr/lib/ckan
ENV DATAPUSHER_HOME /usr/lib/ckan
ENV DATAPUSHER_VENV $DATAPUSHER_HOME/datapusher
ENV DATAPUSHER_CONFIG /etc/ckan
# Build-time variables specified by docker-compose.yml / .env
ARG CKAN_SITE_URL
# Setup virtual environment for DATAPUSHER
RUN mkdir -p $DATAPUSHER_VENV $DATAPUSHER_CONFIG && \
    virtualenv --no-site-packages $DATAPUSHER_VENV && \
    ln -s $DATAPUSHER_VENV/bin/pip /usr/local/bin/ckan-pip &&\
    ln -s $DATAPUSHER_VENV/bin/ckan /usr/local/bin/ckan
# Virtual environment binaries/scripts to be used first
ENV PATH=${DATAPUSHER_VENV}/bin:${PATH}  

# Setup DATAPUSHER
RUN cd $DATAPUSHER_VENV && git clone https://github.com/ckan/datapusher && cd datapusher && ckan-pip install -r requirements.txt && ckan-pip install -e .

RUN chown www-data:www-data -R /var/log/apache2
RUN chown www-data:www-data -R /usr/lib/ckan
RUN a2dissite 000-default
COPY ./apache2/sites-available/datapusher.conf /etc/apache2/sites-enabled/datapusher.conf

CMD ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]
