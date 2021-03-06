## Based on work by https://github.com/earthlab/dockerfiles/blob/master/r-greta/Dockerfile
## https://github.com/rocker-org/ml
## rocker
## kailicht/rbase_keras_tf:1.0
## ingnazio

## FROM nvcr.io/nvidia/cuda:10.0-cudnn7-runtime-ubuntu18.04
FROM tensorflow/tensorflow:latest-gpu
MAINTAINER "Jess Aunsbjørn" thor.vall@gmail.com

RUN echo 'debconf debconf/frontend select Noninteractive' | debconf-set-selections

## Prepare R installation from 
RUN sh -c 'echo "deb https://cloud.r-project.org/bin/linux/ubuntu bionic-cran35/" >> /etc/apt/sources.list' \
    && apt-key adv --keyserver keyserver.ubuntu.com --recv-keys E298A3A825C0D65DFD57CBB651716619E084DAB9 

RUN apt-get update \
    && apt-get upgrade -y -q \
    && apt-get install -y --no-install-recommends \
           libapparmor1 \
           r-base \
           r-base-dev \
           littler \
           r-cran-littler \
           libxml2-dev \
           libxt-dev \
           libssl-dev \
           libcurl4-openssl-dev \
           imagemagick \
           python3-pip \
	         build-essential \
	         curl \
           libfreetype6-dev \
           libzmq3-dev \
           pkg-config \
           python3 \
           python3-dev \
           rsync \
           software-properties-common \
           unzip \
           htop \
           python3.setuptools \
           apt-utils \
           file \
           git \
           libedit2 \
           lsb-release \
           psmisc \
           sudo \
           wget \
	   python3-venv \
  && Rscript -e "install.packages(c('littler', 'docopt'), repo = 'https://cloud.r-project.org')" \
  && ln -s /usr/local/lib/R/site-library/littler/examples/install2.r /usr/local/bin/install2.r \
  && ln -s /usr/local/lib/R/site-library/littler/examples/installGithub.r /usr/local/bin/installGithub.r \
  && ln -s /usr/local/lib/R/site-library/littler/bin/r /usr/local/bin/r \
        && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* \
    && echo 'options(repos = c(CRAN = "https://cloud.r-project.org"))' >> /etc/R/Rprofile.site 

## RUN curl -O https://bootstrap.pypa.io/get-pip.py && \
##     python get-pip.py && \
##    rm get-pip.py
## RUN apt-get install python3-setuptools 

RUN install2.r --error \
    devtools \
    rstan \
    tictoc \
    bayesplot \
    igraph \
    DiagrammeR \
    rgexf \
    XML \
    tidyverse 
	
RUN pip3 --no-cache-dir install \
        ipykernel \
        jupyter \
        matplotlib \
        numpy \
        scipy \ 
        numba \  
        virtualenv \ 
        && \
     python3 -m ipykernel.kernelspec

    
RUN R -e "devtools::install_github('rstudio/tensorflow')"
## RUN R -e "devtools::install_github('rstudio/keras')"
RUN R -e "devtools::install_github('greta-dev/greta')"
 
## Need to configure non-root user for RStudio
RUN useradd rstudio \
  && echo "rstudio:rstudio" | chpasswd \
	&& mkdir /home/rstudio \
	&& chown rstudio:rstudio /home/rstudio \
	&& addgroup rstudio staff

RUN  wget -O libssl1.0.0.deb http://ftp.debian.org/debian/pool/main/o/openssl/libssl1.0.0_1.0.1t-1+deb8u8_amd64.deb \
  && dpkg -i libssl1.0.0.deb \
  && rm libssl1.0.0.deb \
  && RSTUDIO_LATEST=$(wget --no-check-certificate -qO- https://s3.amazonaws.com/rstudio-server/current.ver) \
  && wget -q http://download2.rstudio.org/rstudio-server-${RSTUDIO_LATEST}-amd64.deb \
  && dpkg -i rstudio-server-${RSTUDIO_LATEST}-amd64.deb \
  && rm rstudio-server-*-amd64.deb \
  ## Symlink pandoc & standard pandoc templates for use system-wide
  && ln -s /usr/lib/rstudio-server/bin/pandoc/pandoc /usr/local/bin \
  && ln -s /usr/lib/rstudio-server/bin/pandoc/pandoc-citeproc /usr/local/bin \
  && git clone https://github.com/jgm/pandoc-templates \
  && mkdir -p /opt/pandoc/templates \
  && cp -r pandoc-templates*/* /opt/pandoc/templates && rm -rf pandoc-templates* \
  && mkdir /root/.pandoc && ln -s /opt/pandoc/templates /root/.pandoc/templates \
  && apt-get clean \
  && rm -rf /var/lib/apt/lists/ \
  ## RStudio wants an /etc/R, will populate from $R_HOME/etc
  && mkdir -p /etc/R \
  && mkdir /usr/local/lib/R/etc/ \
  ## Write config files in $R_HOME/etc
  && echo '\n\
    \n# Configure httr to perform out-of-band authentication if HTTR_LOCALHOST \
    \n# is not set since a redirect to localhost may not work depending upon \
    \n# where this Docker container is running. \
    \nif(is.na(Sys.getenv("HTTR_LOCALHOST", unset=NA))) { \
    \n  options(httr_oob_default = TRUE) \
    \n}' >> /usr/local/lib/R/etc/Rprofile.site \
  && echo "PATH=${PATH}" >> /usr/local/lib/R/etc/Renviron \
  ## Need to configure non-root user for RStudio
  #&& useradd rstudio \
  #&& echo "rstudio:rstudio" | chpasswd \
	#&& mkdir /home/rstudio \
	#&& chown rstudio:rstudio /home/rstudio \
	#&& addgroup rstudio staff \
  ## Prevent rstudio from deciding to use /usr/bin/R if a user apt-get installs a package
  &&  echo 'rsession-which-r=/usr/bin/R' >> /etc/rstudio/rserver.conf \
  ## use more robust file locking to avoid errors when using shared volumes:
  && echo 'lock-type=advisory' >> /etc/rstudio/file-locks \ 
  ## configure git not to request password each time 
  && git config --system credential.helper 'cache --timeout=3600' \
  && git config --system push.default simple \
  ## Set up S6 init system
  && wget -P /tmp/ https://github.com/just-containers/s6-overlay/releases/download/v1.11.0.1/s6-overlay-amd64.tar.gz \
  && tar xzf /tmp/s6-overlay-amd64.tar.gz -C / \
  && mkdir -p /etc/services.d/rstudio \
  && echo '#!/usr/bin/with-contenv bash \
           \n exec /usr/lib/rstudio-server/bin/rserver --server-daemonize 0' \
           > /etc/services.d/rstudio/run \
   && echo '#!/bin/bash \
           \n rstudio-server stop' \
           > /etc/services.d/rstudio/finish

COPY userconf.sh /etc/cont-init.d/userconf
# running with "-e ADD=shiny" adds shiny server
COPY add_shiny.sh /etc/cont-init.d/add

COPY pam-helper.sh /usr/lib/rstudio-server/bin/pam-helper

EXPOSE 8787

RUN apt-get install python3-venv

USER rstudio

RUN  python3 -m venv /home/rstudio/.virtualenvs/r-tensorflow

## RUN R -e "tensorflow::install_tensorflow(version = 'gpu', extra_packages = 'tensorflow-probability')"

RUN . /home/rstudio/.virtualenvs/r-tensorflow/bin/activate r-tensorflow \
  && pip install -U pip \
  && pip install -U tensorflow-gpu tensorflow-probability keras

USER root

RUN apt-get install python-pip

CMD ["/init"]
