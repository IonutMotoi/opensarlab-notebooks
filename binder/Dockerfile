FROM oraclelinux:8

LABEL author="ASF" \
    description="Jupyter image based on OPERA ADT dockerfile. version 0.3-gamma: RTC gamma release R3" \
    version="0.3-gamma"

ARG NB_USER=jovyan
ARG NB_UID=1000
ENV USER ${NB_USER}
ENV NB_UID ${NB_UID}
ENV HOME /home/${NB_USER}

USER root

RUN adduser --disabled-password \
    --gecos "Default user" \
    --uid ${NB_UID} \
    ${NB_USER}

RUN python3 -m pip install --no-cache-dir notebook jupyterlab

RUN yum -y update &&\
    yum -y install curl

RUN mkdir -p ${HOME}/OPERA/RTC

RUN chmod -R 755 ${HOME} &&\
    chown -R ${NB_USER}:${NB_USER} ${HOME}/OPERA

# Make sure the contents of our repo are in ${HOME}
COPY . ${HOME}
RUN chown -R ${NB_UID} ${HOME}

USER ${NB_USER}

RUN mkdir -p ${HOME}/OPERA/RTC

ENV CONDA_PREFIX=${HOME}/miniconda3

# install Miniconda
WORKDIR ${HOME}
RUN curl -sSL https://repo.continuum.io/miniconda/Miniconda3-latest-Linux-x86_64.sh -o miniconda.sh &&\
    bash miniconda.sh -b -p ${CONDA_PREFIX} &&\
    rm ${HOME}/miniconda.sh

ENV PATH=${CONDA_PREFIX}/bin:${PATH}
RUN ${CONDA_PREFIX}/bin/conda init bash

RUN curl -sSL https://github.com/opera-adt/RTC/archive/refs/tags/v0.3.tar.gz -o rtc.tar.gz &&\
    tar -xvf rtc.tar.gz -C OPERA &&\
    ln -s OPERA/RTC-0.3 OPERA/RTC &&\
    rm rtc.tar.gz &&\
    python -m pip install ./OPERA/RTC

# copy RTC source code and set rtc_user as owner
COPY --chown=${NB_UID}:${NB_UID} . ${HOME}/OPERA/RTC

# create CONDA environment
RUN conda create --name "RTC" --file /home/rtc_user/OPERA/RTC/Docker/specfile.txt

SHELL ["conda", "run", "-n", "RTC", "/bin/bash", "-c"]

WORKDIR ${HOME}/OPERA

# installing OPERA s1-reader
RUN curl -sSL https://github.com/opera-adt/s1-reader/archive/refs/tags/v0.1.6.tar.gz -o s1_reader_src.tar.gz &&\
    tar -xvf s1_reader_src.tar.gz &&\
    ln -s s1-reader-0.1.6 s1-reader &&\
    rm s1_reader_src.tar.gz &&\
    python -m pip install ./s1-reader

# installing OPERA RTC
RUN python -m pip install ./RTC &&\
    echo "conda activate RTC" >> ${HOME}/.bashrc

WORKDIR ${HOME}/scratch

# ENTRYPOINT ["conda", "run", "--no-capture-output", "-n", "RTC"]