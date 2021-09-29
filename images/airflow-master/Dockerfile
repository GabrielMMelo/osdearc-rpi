FROM raspbian/stretch:latest

RUN apt update

RUN apt upgrade -y

RUN set -ex \
    && buildDeps=' \
        freetds-dev \
        libkrb5-dev \
        libsasl2-dev \
        libssl-dev \
        libffi-dev \
        libpq-dev \
        git \
    '

RUN apt install -yqq --no-install-recommends \
        $buildDeps \
        freetds-bin \
        build-essential \
        default-libmysqlclient-dev \
        apt-utils \
        curl \
        rsync \
        netcat \
        locales \
        tk-dev \
        libncurses5-dev \
        libncursesw5-dev \
        libreadline6-dev \
        libdb5.3-dev \
        libgdbm-dev \
        libsqlite3-dev \
        libssl-dev \
        libbz2-dev \
        libexpat1-dev \
        liblzma-dev \
        zlib1g-dev \
        libffi-dev \
	libpq-dev \
        gfortran \
        libopenblas-dev \
        liblapack-dev

RUN sed -i 's/^# en_US.UTF-8 UTF-8$/en_US.UTF-8 UTF-8/g' /etc/locale.gen \
    && locale-gen \
    && update-locale LANG=en_US.UTF-8 LC_ALL=en_US.UTF-8 \
    && useradd -ms /bin/bash -d /home/airflow airflow

RUN wget https://www.python.org/ftp/python/3.8.0/Python-3.8.0.tar.xz \
    && tar xf Python-3.8.0.tar.xz \
    && cd Python-3.8.0 \
    && ./configure --enable-optimizations --prefix=/usr/local/opt/python-3.8.0 --with-ensurepip=install \
    && make -j 4 \
    && make altinstall \
    && cd .. \
    && rm -r Python-3.8.0 \
    && rm Python-3.8.0.tar.xz \
    && . ~/.bashrc

ENV CRYPTOGRAPHY_DONT_BUILD_RUST=1

RUN ln -s /usr/local/opt/python-3.8.0/bin/python3.8 /usr/bin/python \
    && ln -s /usr/local/opt/python-3.8.0/bin/pip3.8 /usr/bin/pip

USER airflow

RUN pip install -U --user pip setuptools wheel \
    && pip install -U --user pytz \
    && pip install -U --user pyOpenSSL \
    && pip install -U --user ndg-httpsclient \
    && pip install -U --user pyasn1

RUN pip install --user apache-airflow==2.1.4 \
    && pip install --user 'redis==3.2'

RUN pip install -U --user pip setuptools wheel

RUN pip install --user cython

COPY requirements-airflow.txt .

RUN pip install --user -r requirements-airflow.txt

RUN if [ -n "${PYTHON_DEPS}" ]; then pip install --user ${PYTHON_DEPS}; fi

USER root

RUN apt purge --auto-remove -yqq $buildDeps \
    && apt autoremove -yqq --purge \
    && apt clean \
    && rm -rf \
        /var/lib/apt/lists/* \
        /tmp/* \
        /var/tmp/* \
        /usr/share/man \
        /usr/share/doc \
        /usr/share/doc-base

RUN mkdir /opt/airflow
RUN chmod -R 777 /opt/airflow

USER airflow

ENV PATH="/home/airflow/.local/bin:$PATH"
ENV AIRFLOW_CONFIG=/opt/airflow/airflow.cfg
ENV AIRFLOW_HOME=/opt/airflow
