# This Dockerfile contains two images, `builder` and `runtime`.
# `builder` contains all necessary code to build
# `runtime` is stripped down.

FROM debian:buster-slim
LABEL maintainer="Michel Oosterhof <michel@oosterhof.net>"

ENV COWRIE_GROUP=cowrie \
    COWRIE_USER=cowrie \
    COWRIE_HOME=/cowrie

# Set locale to UTF-8, otherwise upstream libraries have bytes/string conversion issues
ENV LC_ALL=en_US.UTF-8 \
    LANG=en_US.UTF-8 \
    LANGUAGE=en_US.UTF-8

RUN groupadd -r -g 1000 ${COWRIE_GROUP} && \
    useradd -r -u 1000 -d ${COWRIE_HOME} -m -g ${COWRIE_GROUP} ${COWRIE_USER}

# Set up Debian prereqs
RUN export DEBIAN_FRONTEND=noninteractive; \
    apt-get update && \
    apt-get install -y \
        -o APT::Install-Suggests=false \
        -o APT::Install-Recommends=false \
      python-pip \
      libssl-dev \
      libffi-dev \
      python-dev \
      python \
      gcc \
      git \
      build-essential \
      libsnappy-dev \
      libmpc-dev \
      python3-distutils \
      nano \
      libssl1.1 \
      libffi6 \
      procps \
      default-libmysqlclient-dev && \
    rm -rf /var/lib/apt/lists/*

RUN chown -R ${COWRIE_USER}:${COWRIE_GROUP} ${COWRIE_HOME}

ENV PATH=${COWRIE_HOME}/cowrie-git/bin:${PATH}
ENV STDOUT=yes

# Build a cowrie environment from github master HEAD.
RUN apt-get install python-setuptools

USER ${COWRIE_USER}

RUN git clone https://github.com/cowrie/cowrie ${COWRIE_HOME}/cowrie-git
RUN cd ${COWRIE_HOME}/cowrie-git && git checkout 7c0ab334909ef785e658c8063a217f3b874bf667
RUN pip install --no-cache-dir --upgrade virtualenv
RUN cd ${COWRIE_HOME} && python -m virtualenv cowrie-env
RUN cd ${COWRIE_HOME} && . cowrie-env/bin/activate
RUN pip install --no-cache-dir --upgrade pip
RUN python -m pip install --no-cache-dir --upgrade cffi
RUN python -m pip install --no-cache-dir --upgrade setuptools
RUN python -m pip install --no-cache-dir --upgrade bcrypt
RUN python -m pip install --no-cache-dir --upgrade mysqlclient
RUN sed -i 's/twisted>=15.2.1/twisted==16.5.0/g' ${COWRIE_HOME}/cowrie-git/requirements.txt
RUN python -m pip install --no-cache-dir --upgrade -r ${COWRIE_HOME}/cowrie-git/requirements.txt
RUN sed -i '/MySQL-python/d' ${COWRIE_HOME}/cowrie-git/requirements-output.txt
RUN python -m pip install --no-cache-dir --upgrade -r ${COWRIE_HOME}/cowrie-git/requirements-output.txt

WORKDIR ${COWRIE_HOME}/cowrie-git

# preserve .dist file when etc/ volume is mounted
VOLUME [ "/cowrie/cowrie-git/var", "/cowrie/cowrie-git/etc" ]
RUN mv ${COWRIE_HOME}/cowrie-git/cowrie.cfg.dist ${COWRIE_HOME}/cowrie-git/cowrie.cfg
RUN sed -i 's/activate_venv "cowrie-env"/activate_venv "..\/cowrie-env"/g' ${COWRIE_HOME}/cowrie-git/bin/cowrie
# RUN sed -i 's/file(/open(/' ${COWRIE_HOME}/cowrie-git/cowrie/core/realm.py

ENTRYPOINT [ "cowrie" ]
CMD [ "start", "-n" ]
EXPOSE 2222 2223
