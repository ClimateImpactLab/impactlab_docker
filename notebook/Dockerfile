FROM jupyter/minimal-notebook

USER root
RUN apt-get update \
  && apt-get install -yq --no-install-recommends \
  libfuse-dev \
  fonts-dejavu \
  tzdata \
  gfortran \
  nano \
  fuse \
  vim \
  git \
  keychain \
  gnupg \
  gnupg2 \
  rsync \
  libcurl4-openssl-dev \
  libssl-dev \
  build-essential \ 
  wget

USER $NB_USER

RUN conda update --yes conda
RUN conda install --yes \
    -c defaults \
    -c pyviz/label/dev \
    -c bokeh/channel/dev \
    -c intake \
    -c conda-forge \
    basemap \
    bokeh=0.12.16 \
    cython \
    cytoolz \
    dask=0.18.0 \
    gdal=2.2.4 \
    dask-ml \
    datashader \
    distributed=1.22.0 \
    esmpy \
    fastparquet \
    ipywidgets \
    jupyter \
    jupyterlab=0.32.1 \
    jupyterlab_launcher=0.10.5 \
    jupyter_client \
    holoviews \
    libsodium \
    lz4 \
    matplotlib \
    msgpack-python \
    nb_conda_kernels \
    netcdf4 \
    nomkl \
    numba \
    numcodecs \
    numpy=1.14.5 \
    pandas=0.23.1 \
    python-blosc \
    pyzmq \
    rasterio \
    s3fs \
    scipy \
    scikit-image \
    scikit-learn \
    toolz=0.9.0 \
    tornado=5.0.2 \
    xarray=0.10.7 \
    zarr=2.2.0 \
    zict \
    intake-xarray \
    && conda clean -tipsy

RUN conda install --yes --channel conda-forge/label/dev geopandas

#install R packages
RUN conda install --quiet --yes \
    'rpy2=2.8*' \
    'r-base=3.4.1' \
    'r-irkernel=0.8*' \
    'r-plyr=1.8*' \
    'r-devtools=1.13*' \
    'r-tidyverse=1.1*' \
    'r-shiny=1.0*' \
    'r-rmarkdown=1.8*' \
    'r-forecast=8.2*' \
    'r-rsqlite=2.0*' \
    'r-reshape2=1.4*' \
    'r-nycflights13=0.2*' \
    'r-caret=6.0*' \
    'r-rcurl=1.95*' \
    'r-crayon=1.3*' \
    'r-randomforest=4.6*' \
    'r-htmltools=0.3*' \
    'r-sparklyr=0.7*' \
    'r-htmlwidgets=1.0*' \
    'r-hexbin=1.27*' && \
    fix-permissions $CONDA_DIR && \
    fix-permissions /home/$NB_USER

RUN pip install --upgrade pip

RUN pip install fusepy \
                click \
                google-cloud==0.32.0 \
                google-cloud-storage \
                jedi \
                kubernetes==6.0.0 \
                dask-kubernetes==0.4.0 \
                gcsfs==0.1.0 \
                xesmf \
                wget \
                mapbox \
                impactlab-tools \
                gsutil \
                nbserverproxy==0.8.1 \
                git+https://github.com/xgcm/xgcm \
                --upgrade --no-cache-dir \
                --upgrade-strategy only-if-needed


RUN jupyter labextension install @jupyter-widgets/jupyterlab-manager \
                                 # @jupyterlab/hub-extension \
                                 @pyviz/jupyterlab_pyviz

RUN jupyter serverextension enable --py nbserverproxy --sys-prefix

USER root
COPY prepare.sh /usr/bin/prepare.sh
RUN chmod +x /usr/bin/prepare.sh
RUN mkdir /pre-home && chown -R $NB_USER /pre-home

ENV DASK_CONFIG=/home/$NB_USER/config.yml
COPY config.yml /pre-home
COPY worker-template.yml /pre-home


RUN export GCSFUSE_REPO=gcsfuse-xenial \
  && echo "deb http://packages.cloud.google.com/apt $GCSFUSE_REPO main" | tee /etc/apt/sources.list.d/gcsfuse.list \
  && curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add - \
  && apt-get update \
  && apt-get install gcsfuse \
  && alias googlefuse=/usr/bin/gcsfuse

# clean up
RUN apt-get clean && rm -rf /var/lib/apt/lists/*
RUN conda clean -tipsy

RUN mkdir /gcs && chown -R $NB_USER /gcs
RUN mkdir /opt/


# Add NB_USER to sudo
RUN echo "$NB_USER ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/notebook
RUN sed -ri "s#Defaults\s+secure_path=\"([^\"]+)\"#Defaults secure_path=\"\1:$CONDA_DIR/bin\"#" /etc/sudoers
USER $NB_USER

ENTRYPOINT ["tini", "--", "/usr/bin/prepare.sh"]
CMD ["start.sh jupyter lab"]