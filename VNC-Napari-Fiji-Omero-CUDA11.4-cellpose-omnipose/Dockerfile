# For finding latest versions of the base image see
# https://github.com/SwissDataScienceCenter/renkulab-docker
ARG RENKU_BASE_IMAGE=renku/renkulab-vnc:0.10.1
FROM ${RENKU_BASE_IMAGE}

USER root

# CUDA
ENV CUDA_VERSION=11.4.0
ENV LD_LIBRARY_PATH=/usr/local/nvidia/lib:/usr/local/nvidia/lib64:/usr/local/cuda-11.4/lib64
ENV NVIDIA_VISIBLE_DEVICES=all
ENV NVIDIA_DRIVER_CAPABILITIES=compute,utility
ENV NVIDIA_REQUIRE_CUDA=cuda>=11.4 brand=tesla,driver>=418,driver<419 brand=tesla,driver>=440,driver<441 driver>=450
ENV NCCL_VERSION=2.11.4
ENV LIBRARY_PATH=/usr/local/cuda/lib64/stubs
ENV PATH=/opt/conda/envs/py37/bin:/usr/local/nvidia/bin:/usr/local/cuda/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/opt/conda/bin:/home/jovyan/.local/bin
 
   
RUN apt-get update \
    && apt-get install -y --no-install-recommends gnupg2 curl ca-certificates \
    &&     curl -fsSL https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/7fa2af80.pub | apt-key add - \
    &&     echo "deb https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64 /" > /etc/apt/sources.list.d/cuda.list \
    &&     echo "deb https://developer.download.nvidia.com/compute/machine-learning/repos/ubuntu1804/x86_64 /" > /etc/apt/sources.list.d/nvidia-ml.list \
    &&     rm -rf /var/lib/apt/lists/*

RUN apt-get update \
    && apt-get install -y --no-install-recommends cuda-cudart-11-4=11.4.43-1 cuda-compat-11-4 \
    &&     ln -s cuda-11.4 /usr/local/cuda \
    &&     rm -rf /var/lib/apt/lists/*

RUN echo "/usr/local/nvidia/lib" >> /etc/ld.so.conf.d/nvidia.conf \
    &&     echo "/usr/local/nvidia/lib64" >> /etc/ld.so.conf.d/nvidia.conf

RUN apt-get update \
    && apt-get install -y --no-install-recommends cuda-libraries-11-4=11.4.0-1 libnpp-11-4=11.4.0.33-1 cuda-nvtx-11-4=11.4.43-1 libcublas-11-4=11.5.2.43-1 libcusparse-11-4=11.6.0.43-1 libnccl2=$NCCL_VERSION-1+cuda11.4 \
    &&     apt-mark hold libnccl2 \
    &&     rm -rf /var/lib/apt/lists/*

RUN apt-get update \
    && apt-get install -y --no-install-recommends libtinfo5 libncursesw5 cuda-cudart-dev-11-4=11.4.43-1 cuda-command-line-tools-11-4=11.4.0-1 cuda-minimal-build-11-4=11.4.0-1 cuda-libraries-dev-11-4=11.4.0-1 cuda-nvml-dev-11-4=11.4.43-1 libnpp-dev-11-4=11.4.0.33-1 libnccl-dev=2.11.4-1+cuda11.4 libcublas-dev-11-4=11.5.2.43-1 libcusparse-dev-11-4=11.6.0.43-1 \
    &&     rm -rf /var/lib/apt/lists/*


# CUDNN Installation
ENV CUDNN_VERSION=8.2.2.26

RUN apt-get update \
    && apt-get install -y --no-install-recommends libcudnn8=$CUDNN_VERSION-1+cuda11.4 libcudnn8-dev=$CUDNN_VERSION-1+cuda11.4 \
    &&     rm -rf /var/lib/apt/lists/*

RUN apt-get update && apt-get install -y --no-install-recommends \
        ocl-icd-libopencl1 \
        clinfo && \
    rm -rf /var/lib/apt/lists/*

# OpenCL Installation
RUN mkdir -p /etc/OpenCL/vendors && \
    echo "libnvidia-opencl.so.1" > /etc/OpenCL/vendors/nvidia.icd
    
RUN apt-get update && apt-get install -y --no-install-recommends \
        ffmpeg libsm6 libxext6 && \
    rm -rf /var/lib/apt/lists/*



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


# Add CMAKE
RUN apt-get update \
    && apt-get install -y --no-install-recommends \ 
        cmake gcc-7 g++-7 \
    && rm -rf /var/lib/apt/lists/*

# GNU Compiler needs to be <= 7
RUN update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-7 90 --slave /usr/bin/g++ g++ /usr/bin/g++-7 --slave /usr/bin/gcov gcov /usr/bin/gcov-7


USER ${NB_USER}


RUN jupyter labextension install @jupyter-widgets/jupyterlab-manager


# install the python dependencies
# COPY requirements.txt environment.yml /tmp/
COPY environment.yml /tmp/
RUN conda env update -q -f /tmp/environment.yml && \
#    /opt/conda/bin/pip install -r /tmp/requirements.txt && \
    conda clean -y --all && \
    conda env export -n "py37" && \
    /opt/conda/envs/py37/bin/python -m ipykernel install --user --name=py37 && \
    conda init bash && \
    echo "conda activate py37" >> ~/.bashrc 


ENV CMAKE_PREFIX_PATH=${CONDA_PREFIX:-"$(dirname $(which conda))/../"}
ENV PYTORCH_BUILD_VERSION=1.3.1
ENV PYTORCH_BUILD_NUMBER=1
ENV TORCH_CUDA_ARCH_LIST=3.5

RUN cd /tmp && \
    git clone --recursive https://github.com/pytorch/pytorch  && \
    cd /tmp/pytorch && \
    git submodule sync && \
    git submodule update --init --recursive && \
    /opt/conda/envs/py37/bin/python3 setup.py install 

# to download cellpose models
RUN /opt/conda/envs/py37/bin/python3 -c "from cellpose import models;models.Cellpose(model_type='cyto');models.Cellpose(model_type='cyto2');models.Cellpose(model_type='nuclei');models.Cellpose(model_type='bact_omni');models.Cellpose(model_type='cyto2_omni');"

# RENKU_VERSION determines the version of the renku CLI
# that will be used in this image. To find the latest version,
# visit https://pypi.org/project/renku/#history.
ARG RENKU_VERSION=0.16.2

########################################################
# Do not edit this section and do not add anything below

# Install renku from pypi or from github if it's a dev version
RUN if [ -n "$RENKU_VERSION" ] ; then \
        source .renku/venv/bin/activate ; \
        currentversion=$(renku --version) ; \
        if [ "$RENKU_VERSION" != "$currentversion" ] ; then \
            pip uninstall renku -y ; \
            gitversion=$(echo "$RENKU_VERSION" | sed -n "s/^[[:digit:]]\+\.[[:digit:]]\+\.[[:digit:]]\+\(\.dev[[:digit:]]\+\)*\(+g\([a-f0-9]\+\)\)*\(+dirty\)*$/\3/p") ; \
            if [ -n "$gitversion" ] ; then \
                pip install --force "git+https://github.com/SwissDataScienceCenter/renku-python.git@$gitversion" ;\
            else \
                pip install --force renku==${RENKU_VERSION} ;\
            fi \
        fi \
    fi

########################################################