FROM ubuntu:20.04 AS base

ENV DEBIAN_FRONTEND=noninteractive


RUN apt-get update -y && \
    apt-get install -y --no-install-recommends \
    ca-certificates \
    git \
    gosu \
    lxterminal \
    net-tools \
    openbox \
    procps \
    python3 \
    python3-pip \
    supervisor \
    tigervnc-standalone-server \
    unzip \
    wget \
    xdg-utils && \
    rm -rf /var/lib/apt/lists && \
    mkdir -p /usr/share/desktop-directories

RUN mkdir -p /usr/share/novnc && \
    cd /usr/share/novnc && \
    git clone https://github.com/novnc/noVNC.git && \
    cd noVNC && \
    git checkout 7485e82b72d4d1356d95ecca2d109cbf49908b9d && \
    mv * /usr/share/novnc/

COPY config/index.html /usr/share/novnc

RUN cd /usr/share/novnc/utils && git clone https://github.com/novnc/websockify websockify

RUN mkdir -p /tmp/.X11-unix
RUN chmod 1777 /tmp/.X11-unix

ENV SC_USER_NAME=scu
RUN groupadd --gid 8004 ${SC_USER_NAME} && \
    useradd --home-dir /home/${SC_USER_NAME} --shell /bin/bash --uid 8004 --gid 8004 ${SC_USER_NAME} && \
    mkdir -p /home/${SC_USER_NAME}

ENV DY_SIDECAR_PATH_INPUTS=/home/${SC_USER_NAME}/work/inputs
ENV DY_SIDECAR_PATH_OUTPUTS=/home/${SC_USER_NAME}/work/outputs

EXPOSE 12000
EXPOSE 8080

ENV LC_ALL=C.UTF-8
ENV LANG=C.UTF-8

FROM base AS production


# Install AxonDeepSec from doc instructions, via miniconda: https://axondeepseg.readthedocs.io/en/latest/documentation.html#installation
WORKDIR $HOME

# Change path to include conda commands
ENV PATH="/opt/miniconda3/bin:${PATH}"
ARG PATH="/opt/miniconda3/bin:${PATH}"

ENV CONDA_DIR=/opt/miniconda3/

# This are libraries needed for napari
RUN apt-get update -y && \
    apt-get install -y --no-install-recommends \
    curl  \
    mesa-utils \
    libgl1-mesa-glx \
    libglib2.0-0 \
    libfontconfig1 \
    libxrender1 \
    libdbus-1-3 \
    libxkbcommon-x11-0 \
    libxi6 \
    libxcb-icccm4 \
    libxcb-image0 \
    libxcb-keysyms1 \
    libxcb-randr0 \
    libxcb-render-util0 \
    libxcb-xinerama0 \
    libxcb-xinput0 \
    libxcb-xfixes0 \
    libxcb-shape0 && \
    rm -rf /var/lib/apt/lists

RUN curl \
    https://repo.anaconda.com/miniconda/Miniconda3-py38_23.10.0-1-Linux-x86_64.sh -o miniconda.sh && \
    bash miniconda.sh -b -p ${CONDA_DIR} \
    && rm -f Miniconda3-latest-Linux-x86_64.sh

RUN git clone https://github.com/neuropoly/axondeepseg.git && \
     cd axondeepseg && \
     conda init bash && \
     conda env create
     
SHELL ["conda", "run", "-n", "ads_venv", "/bin/bash", "-c" ]

RUN  cd axondeepseg && \
     #conda install -c conda-forge  xpra && \
     pip install -e . plugins/ && \
     axondeepseg_test && \
     py.test --cov AxonDeepSeg/ --cov-report term-missing



ENV DISPLAY=:0
ENV XPRA_PORT=9876
EXPOSE 9876

# Activate the conda env at startup
RUN echo "source $CONDA_DIR/bin/activate ads_venv" >> ~/.bashrc

# -----------------------------------
COPY config/menu.xml /etc/xdg/openbox/
COPY config/supervisord.conf /etc/

# Overwrite openbox config to start full screen
COPY config/rc.xml /etc/xdg/openbox/rc.xml
## working directory and permissions

RUN
WORKDIR /home/scu/work
COPY docker/ /docker

CMD ["sh", "-c", "/docker/boot.sh"]
