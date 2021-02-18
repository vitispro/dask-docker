FROM jupyter/base-notebook:lab-3.0.5

USER root

RUN apt-get update \
    && apt-get install -yq --no-install-recommends graphviz git \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

USER $NB_USER

RUN conda install --yes \
    -c conda-forge \
    python-blosc \
    cytoolz \
    dask==2021.2.0 \
    lz4 \
    nomkl \
    numpy==1.18.1 \
    pandas==1.0.1 \
    ipywidgets \
    dask-labextension>=5 \
    python-graphviz \
    && conda clean -tipsy \
    && jupyter lab clean \
    && jlpm cache clean \
    && npm cache clean --force \
    && find /opt/conda/ -type f,l -name '*.a' -delete \
    && find /opt/conda/ -type f,l -name '*.pyc' -delete \
    && find /opt/conda/ -type f,l -name '*.js.map' -delete \
    && find /opt/conda/lib/python*/site-packages/bokeh/server/static -type f,l -name '*.js' -not -name '*.min.js' -delete \
    && rm -rf /opt/conda/pkgs

USER root

# Create the /opt/app directory, and assert that Jupyter's NB_UID/NB_GID values
# haven't changed.
RUN mkdir /opt/app \
    && if [ "$NB_UID" != "1000" ] || [ "$NB_GID" != "100" ]; then \
    echo "Jupyter's NB_UID/NB_GID changed, need to update the Dockerfile"; \
    exit 1; \
    fi

# Copy over the example as NB_USER. Unfortuantely we can't use $NB_UID/$NB_GID
# in the `--chown` statement, so we need to hardcode these values.
COPY --chown=1000:100 examples/ /home/$NB_USER/examples
COPY prepare.sh /usr/bin/prepare.sh

USER root
RUN apt-get update \
    && apt-get -y install curl \
    && apt-get install -y gnupg2

USER root
RUN pip install seaborn \
    && pip install gcsfs \
    && pip install matplotlib

USER root
RUN echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] http://packages.cloud.google.com/apt cloud-sdk main" | tee -a /etc/apt/sources.list.d/google-cloud-sdk.list && curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key --keyring /usr/share/keyrings/cloud.google.gpg  add - && apt-get update -y && apt-get install google-cloud-sdk -y

USER root
RUN conda install -c conda-forge ipywidgets \
    && pip install google-api-python-client httplib2 \
    && pip install git+https://github.com/dask/dask-cloudprovider.git@fix/on_host_maintenance

ENTRYPOINT ["tini", "--", "/usr/bin/prepare.sh"]
