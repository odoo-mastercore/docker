FROM python:3.6.10-buster
LABEL maintainer="far@sinapsys.global"

SHELL ["/bin/bash", "-xo", "pipefail", "-c"]

# Generate locale C.UTF-8 for postgres and general locale data
ENV LANG C.UTF-8

# Install some deps, lessc and less-plugin-clean-css, python3-x.. and wkhtmltopdf
RUN apt-get update \
        && apt-get install -y --no-install-recommends \
            ca-certificates \
            curl \
            dirmngr \
            fonts-noto-cjk \
            gnupg \
            libssl-dev \
            node-less \
            npm \
            xz-utils \
            wget \
            libzlcore-dev \
            libsasl2-dev \
            libldap2-dev \
            nano \
            locate \
            git \
            gcc \
            apt-utils \
            python3-wheel \
            python3-pip \
            swig \
        && curl -o wkhtmltox.deb -sSL https://github.com/wkhtmltopdf/wkhtmltopdf/releases/download/0.12.5/wkhtmltox_0.12.5-1.stretch_amd64.deb \
        && echo '7e35a63f9db14f93ec7feeb0fce76b30c08f2057 wkhtmltox.deb' | sha1sum -c - \
        && apt-get install -y --no-install-recommends ./wkhtmltox.deb \
        && rm -rf /var/lib/apt/lists/* wkhtmltox.deb

# install latest postgresql-client
RUN echo 'deb http://apt.postgresql.org/pub/repos/apt/ buster-pgdg main' > /etc/apt/sources.list.d/pgdg.list \
        && GNUPGHOME="$(mktemp -d)" \
        && export GNUPGHOME \
        && repokey='B97B0AFCAA1A47F044F244A07FCC7D46ACCC4CF8' \
        && gpg --batch --keyserver keyserver.ubuntu.com --recv-keys "${repokey}" \
        && gpg --batch --armor --export "${repokey}" > /etc/apt/trusted.gpg.d/pgdg.gpg.asc \
        && gpgconf --kill all \
        && rm -rf "$GNUPGHOME" \
        && apt-get update  \
        && apt-get install --no-install-recommends -y postgresql-client \
        && rm -f /etc/apt/sources.list.d/pgdg.list \
        && rm -rf /var/lib/apt/lists/*

# Install rtlcss (on Debian buster) and remove openssl
RUN npm install -g rtlcss \
        && dpkg -r --force-depends openssl

# Installing Odoo 13.0 python requirements, adding odoo user
COPY ./requirements.txt /
RUN pip3 install -r /requirements.txt \
        && rm -rf /root/.cache/pip \
        && adduser --system --group --home /home/odoo --shell /bin/bash odoo \
        && rm /requirements.txt

# Install openSSL 1.0.1
RUN cd /usr/local/src/ \
        && wget https://www.openssl.org/source/old/1.0.1/openssl-1.0.1k.tar.gz \
        && tar -xvf /usr/local/src/openssl-1.0.1k.tar.gz \
        && cd /usr/local/src/openssl-1.0.1k \
        && ./config --prefix=/usr/local/ --openssldir=/usr/local/openssl \
        && make \
        && make install \
        && cp -p /usr/local/bin/openssl /usr/bin/openssl \
        && rm -rf /usr/local/src/*


ENV ODOO_VERSION 13.0
USER odoo
RUN cd && git clone --depth 1 --branch ${ODOO_VERSION} https://github.com/odoo/odoo 

USER root
ENV PATH "$PATH:/home/odoo/odoo"

# Copy entrypoint script and Odoo configuration file
COPY ./entrypoint.sh /
COPY ./odoo.conf /etc/odoo/

# Set permissions and Mount /var/lib/odoo to allow restoring filestore and /mnt/extra-addons for users addons
RUN chown odoo /etc/odoo/odoo.conf \
    && mkdir -p /mnt/extra-addons \
    && chown -R odoo /mnt/extra-addons \
    && mkdir -p /usr/local/lib/python3.6/site-packages/pyafipws/cache \
    && chown -R odoo /usr/local/lib/python3.6/site-packages/pyafipws \
    && mkdir -p /var/lib/odoo \
    && chown -R odoo /var/lib/odoo \
    && mkdir -p /var/log/odoo \
    && chown -R odoo /var/log/odoo \
    && touch /var/log/odoo/odoo.log \
    && chown -R odoo /var/log/odoo \
    && mkdir -p /usr/share/GeoIP

VOLUME ["/var/lib/odoo", "/mnt/extra-addons"]

# Expose Odoo services
EXPOSE 8069 8071 8072

# Set the default config file
ENV ODOO_RC /etc/odoo/odoo.conf

COPY wait-for-psql.py /usr/local/bin/wait-for-psql.py
COPY GeoLite2-City.mmdb /usr/share/GeoIP/GeoLite2-City.mmdb

# Set default user when running the container
USER odoo

ENTRYPOINT ["/entrypoint.sh"]
CMD ["odoo-bin"]
