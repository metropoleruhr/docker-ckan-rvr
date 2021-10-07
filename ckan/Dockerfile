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
ENV CKAN_HOME /usr/lib/ckan
ENV CKAN_VENV $CKAN_HOME/default
ENV CKAN_CONFIG /etc/ckan
ENV CKAN_STORAGE_PATH=/var/lib/ckan
# Build-time variables specified by docker-compose.yml / .env
ARG CKAN_SITE_URL
# Setup virtual environment for CKAN
RUN mkdir -p $CKAN_VENV $CKAN_CONFIG $CKAN_STORAGE_PATH && \
    virtualenv --no-site-packages $CKAN_VENV && \
    ln -s $CKAN_VENV/bin/pip /usr/local/bin/ckan-pip &&\
    ln -s $CKAN_VENV/bin/ckan /usr/local/bin/ckan
RUN /usr/local/bin/ckan-pip install pairtree 
RUN /usr/local/bin/ckan-pip install pylibmc
# Virtual environment binaries/scripts to be used first
ENV PATH=${CKAN_VENV}/bin:${PATH}  

# Setup CKAN
#ADD . $CKAN_VENV/src/ckan/
RUN ckan-pip install -U pip && \
    ckan-pip install -e 'git+https://github.com/ckan/ckan.git@ckan-2.9.2#egg=ckan[requirements]' && \
    ckan-pip install --no-cache-dir -r $CKAN_VENV/src/ckan/requirement-setuptools.txt && \
    ckan-pip install --no-cache-dir -r $CKAN_VENV/src/ckan/requirements-py2.txt && \
    ckan-pip install -e $CKAN_VENV/src/ckan/ && \
    chown -R www-data:www-data $CKAN_HOME $CKAN_VENV $CKAN_CONFIG $CKAN_STORAGE_PATH

# Install Extension
RUN ckan-pip install -e "git+https://github.com/ckan/ckanext-spatial.git@v0.0.1#egg=ckanext-spatial"
RUN ckan-pip install --no-cache-dir -r $CKAN_VENV/src/ckanext-spatial/pip-requirements.txt
RUN ckan-pip install -e "git+https://github.com/ckan/ckanext-harvest.git@v1.3.3#egg=ckanext-harvest"
RUN ckan-pip install --no-cache-dir -r $CKAN_VENV/src/ckanext-harvest/pip-requirements.txt
RUN ckan-pip install -e "git+https://github.com/ckan/ckanext-dcat.git@v1.1.1#egg=ckanext-dcat"
RUN ckan-pip install --no-cache-dir -r $CKAN_VENV/src/ckanext-dcat/requirements.txt
RUN cd $CKAN_VENV/src/ && git clone -b 3.8.0 https://github.com/GovDataOfficial/ckanext-dcatde.git
COPY ./ckanext-dcatde/base-requirements.txt $CKAN_VENV/src/ckanext-dcatde/
RUN ckan-pip install $CKAN_VENV/src/ckanext-dcatde
RUN cat $CKAN_VENV/src/ckanext-dcatde/base-requirements.txt
RUN ckan-pip install --no-cache-dir -r $CKAN_VENV/src/ckanext-dcatde/base-requirements.txt
RUN ckan-pip install -e "git+https://github.com/ckan/ckanext-geoview.git#egg=ckanext-geoview"
RUN ckan-pip install --no-cache-dir -r $CKAN_VENV/src/ckanext-geoview/pip-requirements.txt
RUN ckan-pip install -e "git+https://github.com/ckan/ckanext-mapviews.git#egg=ckanext-mapviews"
RUN ckan-pip install -e "git+https://github.com/ckan/ckanext-pages.git@v0.3.1#egg=ckanext-pages"
RUN ckan-pip install --no-cache-dir -r $CKAN_VENV/src/ckanext-pages/requirements.txt
RUN ckan-pip install -e "git+https://github.com/geopython/pycsw.git@2.6.0#egg=pycsw"
RUN ckan-pip install ckanext-pdfview
RUN ckan-pip install -e "git+https://github.com/metropoleruhr/ckanext-rvr#egg=ckanext-rvr"

RUN chown www-data:www-data -R /var/log/apache2
RUN chown www-data:www-data -R /usr/lib/ckan
RUN a2dissite 000-default
COPY ./apache2/sites-available/ckan_default.conf /etc/apache2/sites-enabled/ckan_default.conf

CMD ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]
