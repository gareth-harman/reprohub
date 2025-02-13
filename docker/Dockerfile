FROM golang:1.17.7 as builder

ARG VERSION="1.0.0"

WORKDIR $GOPATH/src/github.com/apptainer
RUN apt-get update && apt-get install -y \
    build-essential \
    uuid-dev \
    libgpgme-dev \
    squashfs-tools \
    libseccomp-dev \
    wget \
    pkg-config \
    git \
    cryptsetup-bin
RUN wget https://github.com/apptainer/apptainer/releases/download/v${VERSION}/apptainer-${VERSION}.tar.gz && \
    tar -xzf apptainer-${VERSION}.tar.gz && \
    cd apptainer-${VERSION} && \
    ./mconfig --prefix=/opt/apptainer && \
    make -C ./builddir && \
    make -C ./builddir install

FROM jupyter/datascience-notebook:2022-03-02

USER root
COPY --from=builder /opt/apptainer /opt/apptainer
ENV PATH="/opt/apptainer/bin:$PATH"
RUN apt-get update && apt-get install -y ca-certificates libseccomp2 \
   squashfs-tools fuse s3fs netbase less parallel tmux screen vim htop \
   && rm -rf /tmp/*

RUN curl --silent --show-error "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" \
  -o "awscliv2.zip" && unzip awscliv2.zip \
  && ./aws/install && rm -rf ./aws awscliv2.zip

RUN wget --quiet https://github.com/kahing/goofys/releases/download/v0.24.0/goofys \
  && chmod +x goofys && mv goofys /usr/local/bin

USER $NB_USER

RUN mamba install --yes datalad rclone 'h5py>3.3=mpi*' ipykernel zarr blosc \
  && wget --quiet https://raw.githubusercontent.com/DanielDent/git-annex-remote-rclone/v0.6/git-annex-remote-rclone \
  && chmod +x git-annex-remote-rclone && mv git-annex-remote-rclone /opt/conda/bin \
  && pip install --no-cache-dir jupytext nbgitpuller datalad-container \
     datalad-osf nibabel nilearn dipy pybids pynidm pydra-ml reproschema nda-tools \
     'pydra>=0.17' plotly itkwidgets==0.32.0 hdf5plugin s3fs datalad-neuroimaging pyontutils \
     PyMARE  gower neurodocker \
  && conda clean --all -f -y && rm -rf /tmp/*

# git+https://github.com/ReproNim/reproman.git@0ff92665

RUN jupyter labextension install @jupyter-widgets/jupyterlab-manager \
  jupyter-matplotlib jupyterlab-datawidgets itkwidgets@0.32.0 jupyterlab-plotly \
  plotlywidget jupyter-threejs --no-build \
  && export NODE_OPTIONS=--max-old-space-size=4096 \
  && jupyter lab build && \
     jupyter lab clean && \
     jlpm cache clean && \
     npm cache clean --force && \
     rm -rf $HOME/.node-gyp && \
     rm -rf $HOME/.local && rm -rf /tmp/*

# Install jupyter server proxy and desktop
USER root

RUN apt-get -y update \
   && apt-get install -y dbus-x11 \
       firefox \
       xfce4 \
       xfce4-panel \
       xfce4-session \
       xfce4-settings \
       xorg \
       xubuntu-icon-theme \
    && rm -rf /tmp/*

# Remove light-locker to prevent screen lock
ARG TURBOVNC_VERSION=2.2.6
RUN wget -q "https://sourceforge.net/projects/turbovnc/files/${TURBOVNC_VERSION}/turbovnc_${TURBOVNC_VERSION}_amd64.deb/download" -O turbovnc_${TURBOVNC_VERSION}_amd64.deb && \
   apt-get install -y -q ./turbovnc_${TURBOVNC_VERSION}_amd64.deb && \
   apt-get remove -y -q light-locker && \
   rm ./turbovnc_${TURBOVNC_VERSION}_amd64.deb && \
   ln -s /opt/TurboVNC/bin/* /usr/local/bin/

# apt-get may result in root-owned directories/files under $HOME
RUN mkdir /opt/extras && chown -R $NB_UID:$NB_GID $HOME /opt/extras

USER $NB_USER

RUN cd /opt/extras && \
    git clone --depth 1 https://github.com/jupyterhub/jupyter-remote-desktop-proxy.git && \
    cd jupyter-remote-desktop-proxy && \
    conda env update -n base --file environment.yml && \
    cd /opt && rm -rf /opt/extras/* && \
    conda clean --all -f -y && rm -rf /tmp/*

