# For finding latest versions of the base image see
# https://github.com/SwissDataScienceCenter/renkulab-docker
ARG RENKU_BASE_IMAGE=renku/renkulab-vnc:0.11.0
FROM ${RENKU_BASE_IMAGE}

# Uncomment and adapt if code is to be included in the image
# COPY src /code/src

# Uncomment and adapt if your R or python packages require extra linux (ubuntu) software
# e.g. the following installs apt-utils and vim; each pkg on its own line, all lines
# except for the last end with backslash '\' to continue the RUN line
#
# USER root
# RUN apt-get update && \
#    apt-get install -y --no-install-recommends \
#    apt-utils \
#    vim
# USER ${NB_USER}

USER root

# Install Fiji
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

# Add simple-omero-client
ENV SOC_VERSION 5.9.0
RUN wget -q https://github.com/GReD-Clermont/simple-omero-client/releases/download/5.9.0/simple-omero-client-${SOC_VERSION}.jar \
        && cp simple-omero-client-${SOC_VERSION}.jar /opt/Fiji.app/plugins \
        && rm simple-omero-client-${SOC_VERSION}.jar \
        && chmod -R a+rwX /opt/Fiji.app/plugins


# Add update-sites
ENV fiji /opt/Fiji.app/ImageJ-linux64
ENV updateCommand  --update update
ENV addCommand  --update add-update-site 

RUN ${fiji} ${updateCommand}

RUN ${fiji} ${addCommand} "PTBIOP" "https://biop.epfl.ch/Fiji-Update"
RUN ${fiji} ${addCommand} "IBMP-CNRS" "https://sites.imagej.net/Mutterer/"
RUN ${fiji} ${addCommand} "3D ImageJ Suite" "https://sites.imagej.net/Tboudier/"
RUN ${fiji} ${addCommand} "IJPB-plugins" "https://sites.imagej.net/IJPB-plugins/"
RUN ${fiji} ${addCommand} "CSBDeep" "https://sites.imagej.net/CSBDeep/"
RUN ${fiji} ${addCommand} "StarDist" "https://sites.imagej.net/StarDist/"

RUN ${fiji} ${updateCommand}

# Install ilastik
ENV ilastik_version 1.3.3
RUN wget -q https://files.ilastik.org/ilastik-${ilastik_version}post3-Linux.tar.bz2 \
      && tar -xf ilastik-${ilastik_version}post3-Linux.tar.bz2 -C /opt/ \
      && rm ilastik-${ilastik_version}post3-Linux.tar.bz2 \
      && chmod -R a+rwX /opt/ilastik-${ilastik_version}post3-Linux

RUN mkdir -p /home/jovyan/Desktop/
COPY ilastik.desktop /home/jovyan/Desktop/
RUN chmod +x /home/jovyan/Desktop/ilastik.desktop


#Insall QuPath
ENV qupath_version 0.3.2
RUN wget -q https://github.com/qupath/qupath/releases/download/v${qupath_version}/QuPath-${qupath_version}-Linux.tar.xz \
      && tar -xf QuPath-${qupath_version}-Linux.tar.xz -C /opt/ \
      && rm QuPath-${qupath_version}-Linux.tar.xz \
      && chmod u+x /opt/QuPath/bin/QuPath \ 
      && chmod u+x /opt/QuPath/bin/QuPath.sh

COPY qupath.desktop /home/jovyan/Desktop/
RUN chmod +x /home/jovyan/Desktop/qupath.desktop

# adds QuPath extensions 
env QP_EXT_DIR /home/jovyan/Desktop/QuPath_Common_Data/extensions/
RUN mkdir -p ${QP_EXT_DIR}
RUN wget -q https://github.com/qupath/qupath-extension-omero/releases/download/v0.3.0/qupath-extension-omero-0.3.0.jar
RUN mv qupath-extension-omero-0.3.0.jar ${QP_EXT_DIR} \ 
    && chmod -R a+rwX ${QP_EXT_DIR} \
    && chmod -R a+rwX ${QP_EXT_DIR}qupath-extension-omero-0.3.0.jar




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
ARG RENKU_VERSION=1.2.4

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
