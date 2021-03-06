##################################
# ESCOMP/CESM-Lab/2.1 Dockerfile #
##################################

# This builds on top of the CESM 2.1 container, and adds the Jupyter Lab environment
FROM escomp/cesm-2.1

# Set up the Conda version - using the pangeo/base-image as a foundation here:
ENV CONDA_VERSION=4.8.3-4 \
    CONDA_ENV=notebook \
    NB_USER=user \
    NB_GROUP=ncar \
    NB_UID=1000 \
    SHELL=/bin/bash \
    LANG=C.UTF-8  \
    LC_ALL=C.UTF-8 \
    CONDA_DIR=/srv/conda

# Additional environment setup that depends on the above:
ENV NB_PYTHON_PREFIX=${CONDA_DIR}/envs/${CONDA_ENV} \
    DASK_ROOT_CONFIG=${CONDA_DIR}/etc \
    HOME=/home/${NB_USER} \
    PATH=${CONDA_DIR}/bin:${PATH}

# Add the Conda init and set permissions on the directory:
# (Could clean this up, and push changes back to Pangeo -- eg, /srv is hardcoded)
RUN sudo /bin/bash -c "echo '. ${CONDA_DIR}/etc/profile.d/conda.sh ; conda activate ${CONDA_ENV}' > /etc/profile.d/init_conda.sh"  && \
    sudo chown -R ${NB_USER}:${NB_GROUP} /srv

# Install miniforge:
RUN URL="https://github.com/conda-forge/miniforge/releases/download/${CONDA_VERSION}/Miniforge3-${CONDA_VERSION}-Linux-x86_64.sh" && \
    wget --quiet ${URL} -O miniconda.sh && \
    /bin/bash miniconda.sh -u -b -p ${CONDA_DIR} && \
    rm miniconda.sh && \
    conda clean -afy && \
    find ${CONDA_DIR} -follow -type f -name '*.a' -delete && \
    find ${CONDA_DIR} -follow -type f -name '*.pyc' -delete

COPY --chown=${NB_USER}:${NB_GROUP} Files/*yml /srv/

RUN mv /srv/condarc.yml ${CONDA_DIR}/.condarc && \
    mv /srv/dask_config.yml ${CONDA_DIR}/etc/dask.yml

RUN sudo yum install -y graphviz 

RUN conda env create --name ${CONDA_ENV} -f /srv/environment.yml  && \
        conda clean -yaf && \
        find ${CONDA_DIR} -follow -type f -name '*.a' -delete && \
        find ${CONDA_DIR} -follow -type f -name '*.pyc' -delete && \
        find ${CONDA_DIR} -follow -type f -name '*.js.map' -delete && \
        find ${NB_PYTHON_PREFIX}/lib/python*/site-packages/bokeh/server/static -follow -type f -name '*.js' ! -name '*.min.js' -delete

RUN pip --no-cache-dir install rechunker


RUN export PATH=${NB_PYTHON_PREFIX}/bin:${PATH} && \
    jupyter labextension install --clean \
         @jupyter-widgets/jupyterlab-manager \
         @jupyterlab/geojson-extension \
         dask-labextension \
         @pyviz/jupyterlab_pyviz \
         jupyter-leaflet && \
    sudo rm -rf /tmp/* && \
    rm -rf ${HOME}/.cache ${HOME}/.npm ${HOME}/.yarn && \
    rm -rf ${NB_PYTHON_PREFIX}/share/jupyter/lab/staging && \
    find ${CONDA_DIR} -follow -type f -name '*.a' -delete && \
    find ${CONDA_DIR} -follow -type f -name '*.pyc' -delete && \
    find ${CONDA_DIR} -follow -type f -name '*.js.map' -delete 

COPY Files/start /srv
RUN sudo chmod +x /srv/start 

COPY Files/cesm_aliases.ipy /etc/ipython/
RUN  sudo /bin/bash -c 'echo "c.InteractiveShellApp.exec_files = [ \"/etc/ipython/cesm_aliases.ipy\" ] " >> /etc/ipython/ipython_config.py'



#EXPOSE 8888
USER user
WORKDIR /home/user
ENV SHELL /bin/bash
ENTRYPOINT ["/srv/start"]
#ENTRYPOINT ["/opt/conda/bin/jupyter", "lab", "--ip=0.0.0.0", "--no-browser", "--NotebookApp.token=''", "--NotebookApp.password=''"]
