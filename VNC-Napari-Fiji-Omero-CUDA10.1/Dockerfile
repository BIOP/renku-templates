# For finding latest versions of the base image see
# https://github.com/SwissDataScienceCenter/renkulab-docker
ARG RENKU_BASE_IMAGE=renku/renkulab-vnc:0.7.7
FROM ${RENKU_BASE_IMAGE}

USER root

ENV CUDA_VERSION=10.1.243
ENV CUDNN_VERSION=7.6.5.32
ENV CUDA_PKG_VERSION=10-1=10.1.243-1
ENV LD_LIBRARY_PATH=/usr/local/nvidia/lib:/usr/local/nvidia/lib64
ENV NVIDIA_VISIBLE_DEVICES=all
ENV NVIDIA_DRIVER_CAPABILITIES=compute,utility
ENV NVIDIA_REQUIRE_CUDA=cuda>=10.1 brand=tesla,driver>=384,driver<385 brand=tesla,driver>=396,driver<397
ENV NCCL_VERSION=2.4.8
ENV LIBRARY_PATH=/usr/local/cuda/lib64/stubs
ENV PATH=/usr/local/nvidia/bin:/usr/local/cuda/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/opt/conda/bin:/home/jovyan/.local/bin

RUN apt-get update \
    && apt-get install -y --no-install-recommends gnupg2 curl ca-certificates \
    &&     curl -fsSL https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/7fa2af80.pub | apt-key add - \
    &&     echo "deb https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64 /" > /etc/apt/sources.list.d/cuda.list \
    &&     echo "deb https://developer.download.nvidia.com/compute/machine-learning/repos/ubuntu1804/x86_64 /" > /etc/apt/sources.list.d/nvidia-ml.list \
    &&     rm -rf /var/lib/apt/lists/*

RUN apt-get update \
    && apt-get install -y --no-install-recommends cuda-cudart-$CUDA_PKG_VERSION cuda-compat-10-1 \
    &&     ln -s cuda-10.1 /usr/local/cuda \
    &&     rm -rf /var/lib/apt/lists/*

RUN echo "/usr/local/nvidia/lib" >> /etc/ld.so.conf.d/nvidia.conf \
    &&     echo "/usr/local/nvidia/lib64" >> /etc/ld.so.conf.d/nvidia.conf

RUN apt-get update \
    && apt-get install -y --no-install-recommends cuda-libraries-$CUDA_PKG_VERSION cuda-nvtx-$CUDA_PKG_VERSION libcublas10=10.2.1.243-1 libnccl2=$NCCL_VERSION-1+cuda10.1 \
    &&     apt-mark hold libnccl2 \
    &&     rm -rf /var/lib/apt/lists/*

RUN apt-get update \
    && apt-get install -y --no-install-recommends cuda-libraries-dev-$CUDA_PKG_VERSION cuda-nvml-dev-$CUDA_PKG_VERSION cuda-minimal-build-$CUDA_PKG_VERSION cuda-command-line-tools-$CUDA_PKG_VERSION libnccl-dev=$NCCL_VERSION-1+cuda10.1 libcublas-dev=10.2.1.243-1\
    &&     rm -rf /var/lib/apt/lists/*
    
RUN apt-get update \    
    && apt-get install -y --no-install-recommends libcudnn7=$CUDNN_VERSION-1+cuda10.1 \
    &&     apt-mark hold libcudnn7 \
    &&     rm -rf /var/lib/apt/lists/*
    
    

# Install ImageJ
RUN wget -q https://downloads.imagej.net/fiji/latest/fiji-linux64.zip \
      && unzip fiji-linux64.zip -d /opt/ \
      && rm fiji-linux64.zip \
      && chmod -R a+rwX /opt/Fiji.app

RUN mkdir -p /home/jovyan/Desktop/
COPY Fiji.desktop /home/jovyan/Desktop/
RUN chmod +x /home/jovyan/Desktop/Fiji.desktop

# Add OMERO plugins
ENV OMEROIJ_VERSION 5.5.19
RUN wget -q https://github.com/ome/omero-insight/releases/download/v${OMEROIJ_VERSION}/omero_ij-${OMEROIJ_VERSION}-all.jar \
        && cp omero_ij-${OMEROIJ_VERSION}-all.jar /opt/Fiji.app/plugins \
        && rm omero_ij-${OMEROIJ_VERSION}-all.jar \
        && chmod -R a+rwX /opt/Fiji.app/plugins
        
RUN apt-get -y update &&\
    apt-get -y install db5.3-util  && \
    apt-get -y install libssl-dev libdb++-dev libdb-dev libbz2-dev libmcpp-dev 

USER ${NB_USER}

# install the python dependencies
COPY requirements.txt environment.yml /tmp/
RUN conda env update -q -f /tmp/environment.yml && \
    /opt/conda/bin/pip install -r /tmp/requirements.txt && \
    conda clean -y --all && \
    conda env export -n "root"

# RENKU_VERSION determines the version of the renku CLI
# that will be used in this image. To find the latest version,
# visit https://pypi.org/project/renku/#history.
ARG RENKU_VERSION=0.15.1

########################################################
# Do not edit this section and do not add anything below

RUN if [ -n "$RENKU_VERSION" ] ; then \
    pipx uninstall renku && \
    pipx install --force renku==${RENKU_VERSION} \
    ; fi

########################################################