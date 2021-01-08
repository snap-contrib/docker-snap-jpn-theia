FROM ubuntu:20.04

USER root

ARG DEBIAN_FRONTEND=noninteractive

ENV LC_ALL=en_US.UTF-8                                                  \
    LANG=en_US.UTF-8                                                    \
    LANGUAGE=en_US.UTF-8                                                \
    SHELL=/bin/bash                                                     \
    PATH=/srv/conda/bin:$PATH                                           \
    NB_USER=jovyan                                                      \
    NB_UID=1002                                                         \
    NB_GID=1002                                                         \
    USER=jovyan                                                         \ 
    HOME=/home/jovyan                                                   \
    CONDA_DIR=/srv/conda/                                               \
    APP_BASE=/srv 
    
RUN groupadd --gid ${NB_GID} ${NB_USER} &&                              \
    useradd --comment "Default user"                                    \
            --create-home                                               \
            --gid ${NB_GID}                                             \
            --no-log-init                                               \
            --shell /bin/bash                                           \
            --uid ${NB_UID} ${NB_USER}

RUN apt-get -qq update &&                                               \
    apt-get -qq install --yes                                           \
    apt-utils                                                           \
    wget                                                                \
    make                                                                \
    g++                                                                 \
    sudo                                                                \
    vim                                                                 \
    tree                                                                \
    file                                                                \
    unzip                                                               \ 
    gnupg-agent                                                         \
    ca-certificates                                                     \
    curl                                                                

RUN wget --quiet -O - https://deb.nodesource.com/gpgkey/nodesource.gpg.key |  apt-key add - && \
    echo "deb https://deb.nodesource.com/node_10.x bionic main" >> /etc/apt/sources.list.d/nodesource.list && \
    echo "deb-src https://deb.nodesource.com/node_10.x bionic main" >> /etc/apt/sources.list.d/nodesource.list

RUN apt-get -qq install --yes --no-install-recommends                   \
    less                                                                \
    nodejs                                                              \
    unzip                                                               \
    git-flow                                                            \
    openjdk-8-jdk                                                       \
    locales > /dev/null &&                                              \
    apt-get -qq purge --yes &&                                          \
    apt-get -qq clean --yes &&                                          \
    rm -rf /var/lib/apt/lists/* &&                                      \
    echo "LC_ALL=en_US.UTF-8" >> /etc/environment &&                    \
    echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen &&                      \ 
    echo "LANG=en_US.UTF-8" > /etc/locale.conf &&                       \
    locale-gen en_US.UTF-8

EXPOSE 8888

ENV NB_PYTHON_PREFIX=${CONDA_DIR}/envs/notebook                         \
    KERNEL_PYTHON_PREFIX=${NB_PYTHON_PREFIX}                            \
    PATH=${NB_PYTHON_PREFIX}/bin:${CONDA_DIR}/bin:${NPM_DIR}/bin:${PATH}\
    PATH=${HOME}/.local/bin:${PATH}                                     \
    CONDA_DEFAULT_ENV=${KERNEL_PYTHON_PREFIX}                           \
    NVM_DIR=$HOME/.nvm

COPY .docker/activate-conda.sh /etc/profile.d/activate-conda.sh

COPY .docker/base-environment.yml /tmp/base-environment.yml

COPY .docker/install-miniforge.bash /tmp/install-miniforge.bash

# miniforge
RUN bash /tmp/install-miniforge.bash &&                                 \
    rm /tmp/install-miniforge.bash /tmp/base-environment.yml

ADD .docker/playground-sudo /etc/sudoers.d/playground-sudo

# workspace
RUN mkdir -p /workspace &&                                              \
    chown -R ${NB_USER}:${NB_GID} /workspace &&                         \
    mkdir -p /usr/local/share/jupyter &&                                \
    chown -R ${NB_USER}:${NB_GID} /usr/local/share/jupyter &&           \
    chmod 440 /etc/sudoers.d/playground-sudo

# setup conda activate/deactivate
#RUN ${CONDA_DIR}/bin/conda init bash && . ${HOME}/.bashrc

RUN mkdir -p ${APP_BASE}/theia &&                                       \
    chown -R ${NB_USER}:${NB_GID} ${APP_BASE}/theia &&                  \
    echo "export PATH=/srv/conda/envs/notebook/bin/:$PATH" >> /root/.bashrc

ENV PATH=/srv/conda/envs/notebook/bin/:$PATH

# jupyter config
ADD .docker/jupyter_notebook_config.py /etc/jupyter/jupyter_notebook_config.py

USER ${NB_USER}

WORKDIR /workspace

# theia
ADD .docker/package.json ${APP_BASE}/theia/package.json

RUN curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.5/install.sh | bash

RUN . "$NVM_DIR/nvm.sh" &&                                              \
    nvm install 12.20.0 &&                                              \
    npm config delete prefix                                         
    
       
RUN . "$NVM_DIR/nvm.sh" &&                                              \
    nvm use --delete-prefix v12.20.0 &&                                 \
    npm install -g yarn &&                                              \
    cd ${APP_BASE}/theia &&                                             \
    THEIA_ELECTRON_SKIP_REPLACE_FFMPEG=1 yarn &&                        \
    cd ${APP_BASE}/theia &&                                             \
    NODE_OPTIONS="--max_old_space_size=4096" &&                         \
    yarn theia build &&                                                 \
    yarn cache clean &&                                                 \
    echo "cd /workspace" >> /home/jovyan/.bashrc

USER ${NB_USER}

COPY --chown=1002:1002 environment.yml /tmp/environment.yml          
    
RUN mamba env create -f /tmp/environment.yml &&                         \
    rm -fr /tmp/environment.yml

