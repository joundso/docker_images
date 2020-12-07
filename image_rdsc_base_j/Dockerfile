FROM ubuntu:20.04

# set environment variable to supress user interaction
ENV DEBIAN_FRONTEND=noninteractive \
    JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64

# set ENV-Vars
ARG DISPLAY
ENV DISPLAY=${DISPLAY}

RUN apt-get update && apt-get install -y --no-install-recommends --no-install-suggests \
    apt-utils\ 
    build-essential \
    byobu \
    cmake \
    curl \
    #default-jdk \
    #default-jre \
    openjdk-8-jdk \
    dialog \
    dirmngr \
    ffmpeg \
    gcc-8 \
    gdebi-core \
    git \
    gpg-agent \
    htop \
    iputils-ping \
    less \
    libcurl4-openssl-dev \
    libssh-dev \
    libssl-dev \
    locales \
    make \
    nano \
    net-tools \
    software-properties-common \
    ssh \
    sudo \
    tar \
    traceroute \
    unzip \
    vim \
    wget
RUN rm -rf /var/lib/apt/lists/*

RUN locale-gen en_US.utf8 \
    && /usr/sbin/update-locale LANG=en_US.UTF-8
ENV LANG=en_US.UTF-8

RUN echo JAVA_HOME="${JAVA_HOME}" >> /etc/environment

# Add R apt repository for latest R
# https://cran.r-project.org/bin/linux/ubuntu/
RUN gpg --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys E298A3A825C0D65DFD57CBB651716619E084DAB9
RUN gpg -a --export E298A3A825C0D65DFD57CBB651716619E084DAB9 | sudo apt-key add -
RUN add-apt-repository "deb http://cran.r-project.org/bin/linux/ubuntu $(lsb_release -cs)-cran40/"

# install required custom system dependencies
RUN apt-get install -y --no-install-recommends --no-install-suggests \
    imagemagick \
    # pocketsphinx
    libasound2-dev \ 
    libboost-dev \
    libboost-system-dev \
    libboost-filesystem-dev \
    libbz2-dev \
    libcairo2-dev \
    libclang-dev \
    libffi-dev \
    libglu1-mesa-dev \
    libgmp3-dev \
    libgsl-dev \
    # rJava dependencies
    liblzma-dev \ 
    libmagick++-dev \
    libmpfr-dev \
    libobjc-7-dev \
    libopenmpi-dev \
    libpq-dev \
    libsasl2-dev \
    # V8 dependencies
    libnode-dev \
    libxt-dev \
    # xml2 package
    libxml2-dev \
    # xslt package
    libxslt1-dev \
    libzmq3-dev \
    lmodern \
    # pocketsphinx
    libpulse-dev \
    locate \
    nano \
    net-tools \
    pandoc \
    pandoc-citeproc \
    # rstudio dependency
    psmisc \
    python3-dev \
    python3-pip \
    python3-venv \
    qpdf \
    r-base-dev \
    # pocketsphinx
    swig \
    # dependencies for aplpack
    tcl8.6-dev \
    texinfo \
    tk8.6-dev \
    traceroute \
    unixodbc-dev

RUN apt-get clean && \ 
    apt-get autoclean && \
    rm -rf /var/lib/apt/lists/*

# add user + password
# https://stackoverflow.com/questions/2150882/how-to-automatically-add-user-account-and-password-with-a-bash-script
# workaround, to get stuff properly installed in users home dir
# set rstudio user here
ENV RSESSION_USER="user" 
ENV USER_UID=1000
ENV USER_GID=${USER_UID}

RUN groupadd --gid ${USER_GID} ${RSESSION_USER} && \
    useradd --uid ${USER_UID} --gid ${USER_GID} -m ${RSESSION_USER}
RUN echo ${RSESSION_USER}:password | chpasswd 
RUN echo ${RSESSION_USER} ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/${RSESSION_USER} && \
    chmod 0440 /etc/sudoers.d/${RSESSION_USER}

# make R site-library default package installation folder writeable
RUN mkdir -p /usr/local/lib/R/site-library
RUN chown -R ${RSESSION_USER}:${RSESSION_USER} /usr/local/lib/R/site-library

# set cran repo
RUN echo "options('repos' = 'https://cloud.r-project.org/')" >> /etc/R/Rprofile.site

# Update where R expects to find various Java files
RUN R CMD javareconf

# first, make sure, that package "remotes" is installed, so we can make use
# of its function remotes::update_packages 
# https://rdrr.io/cran/remotes/man/update_packages.html
RUN R -q -e "install.packages(c('remotes', 'devtools', 'reticulate', 'data.table'), quiet = T)"

# switch user
USER ${RSESSION_USER}

# # install miniconda via reticulate and configure python
# RUN R -q -e "reticulate::install_miniconda(); reticulate::py_config()"

# switch back
# USER root

# # we can now add add the virtualenv python to PATH (on first place)
# # add newly installed dependencies to PATH
# ENV PATH="/home/${RSESSION_USER}/.local/share/r-miniconda/envs/r-reticulate/bin:${PATH}"

# # Add RETICULATE_PYTHON variable to Renviron
# RUN echo "RETICULATE_PYTHON=/home/${RSESSION_USER}/.local/share/r-miniconda/envs/r-reticulate/bin/python" >> /etc/R/Renviron
# # Since R 4.0.0, we also need to add R_LIBS_SITE to /etc/R/Renviron
# ENV R_LIBS_SITE=/usr/local/lib/R/site-library
# RUN echo "R_LIBS_SITE=${R_LIBS_SITE}" >> /etc/R/Renviron

# # switch user (let everything in .virtualenv/r-reticulate be installed by the user)
# # we can use pip here, since we added the virtualenv to the beginning of our PATH variable
# USER ${RSESSION_USER}

# # install python prerequisites
# RUN yes | pip install \
#     setuptools \
#     wheel

USER root

# environment variable for data.table
# https://github.com/Rdatatable/data.table/pull/3435/files
RUN echo "R_DATATABLE_NUM_PROCS_PERCENT=100" >> /etc/R/Renviron