FROM continuumio/miniconda3:4.5.4

# Dumb init
RUN wget -O /usr/local/bin/dumb-init https://github.com/Yelp/dumb-init/releases/download/v1.2.0/dumb-init_1.2.0_amd64
RUN chmod +x /usr/local/bin/dumb-init

RUN conda install --yes \
    -c defaults \
    -c conda-forge \
    bokeh=0.12.16 \
    cloudpickle=0.5.3 \
    cython \
    cytoolz \
    dask=0.18.0 \
    datashader \
    distributed=1.22.0 \
    gdal=2.2.4 \
    esmpy \
    fastparquet \
    git \
    lz4 \
    msgpack-python \
    netcdf4 \
    nomkl \
    numba \
    numcodecs \
    numpy=1.14.5 \
    pandas=0.23.1 \
    python-blosc \
    rasterio \
    s3fs \
    scikit-image \
    toolz=0.9.0 \
    tornado=5.0.2 \
    scipy \
    xarray==0.10.7 \
    zarr==2.2.0 \
    zict 

RUN conda install --yes --channel conda-forge/label/dev geopandas

USER root
RUN apt-get update \
  && apt-get install -yq --no-install-recommends libfuse-dev nano fuse gnupg gnupg2 \
  && apt-get clean \
  && rm -rf /var/lib/apt/lists/*

RUN export GCSFUSE_REPO=gcsfuse-xenial \
  && echo "deb http://packages.cloud.google.com/apt $GCSFUSE_REPO main" | tee /etc/apt/sources.list.d/gcsfuse.list \
  && curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add - \
  && apt-get update \
  && apt-get install gcsfuse \
  && alias googlefuse=/usr/bin/gcsfuse

RUN conda install --yes -c conda-forge setuptools

RUN conda clean -tipsy

RUN pip install click \
                urllib3 \
                fusepy \
                gcsfs==0.1.0 \
                dask-kubernetes==0.4.0 \
                wget \ 
                impactlab-tools \
                climate-toolbox \
                gsutil \
                google-cloud-storage \
                google-cloud==0.32.0 \
                xesmf \
                --upgrade --no-cache-dir --upgrade-strategy only-if-needed

ENV OMP_NUM_THREADS=1
ENV DASK_DISTRIBUTED__ADMIN__TICK__LIMIT=5s

USER root
COPY prepare.sh /usr/bin/prepare.sh
RUN chmod +x /usr/bin/prepare.sh
RUN mkdir /opt/app
RUN mkdir /gcs

ENTRYPOINT ["/usr/local/bin/dumb-init", "/usr/bin/prepare.sh"]