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

# RUN adduser --disabled-password \
#     --gecos "Default user" \
#     --uid ${NB_UID} \
#     ${NB_USER}

RUN adduser --uid ${NB_UID} ${NB_USER}

RUN yum -y update &&\
    yum -y install curl

RUN mkdir -p ${HOME}/OPERA/RTC

RUN chmod -R 755 ${HOME} &&\
    chown -R ${NB_USER}:${NB_USER} ${HOME}/OPERA

# Make sure the contents of our repo are in ${HOME}
# Make sure the contents of our repo are in ${HOME}
COPY . ${HOME}
COPY ./binder/.condarc ${HOME}/.condarc
USER root
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

RUN conda install -n base -c conda-forge jupyterlab jupyter nb_conda_kernels kernda

RUN curl -sSL https://github.com/opera-adt/RTC/archive/refs/tags/v0.3.tar.gz -o rtc.tar.gz &&\
    tar -xvf rtc.tar.gz -C OPERA &&\
    ln -s OPERA/RTC-0.3 OPERA/RTC &&\
    rm rtc.tar.gz &&\
    python -m pip install ./OPERA/RTC-0.3

# # set jovyan as owner of 
# COPY --chown=${NB_UID}:${NB_UID} . ${HOME}/OPERA/RTC

# create CONDA environment
RUN conda create --name notebook --file ${HOME}/OPERA/RTC-0.3/Docker/specfile.txt &&\
    conda install -c conda-forge -n notebook asf_search boto3   
    # conda run -n notebook kernda --display-name notebook -o /home/jovyan/miniconda3/notebook/share/jupyter/kernels/python3/kernel.json


SHELL ["conda", "run", "-n", "notebook", "/bin/bash", "-c"]

# WORKDIR ${HOME}/OPERA

# installing OPERA s1-reader
RUN curl -sSL https://github.com/opera-adt/s1-reader/archive/refs/tags/v0.1.6.tar.gz -o s1_reader_src.tar.gz &&\
    tar -xvf s1_reader_src.tar.gz &&\
    ln -s s1-reader-0.1.6 s1-reader &&\
    rm s1_reader_src.tar.gz &&\
    python -m pip install ./s1-reader

# RUN echo "conda activate notebook" >> ${HOME}/.bashrc

# WORKDIR ${HOME}/scratch

# ENTRYPOINT ["conda", "run", "--no-capture-output", "-n", "RTC"]