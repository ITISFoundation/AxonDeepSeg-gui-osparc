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

# v0.13.0
RUN cd /usr/share/novnc/utils &&\ 
    git clone https://github.com/novnc/websockify websockify &&\ 
    cd websockify && \
    git checkout e81894751365afc19fe64fc9d0e5c6fc52655c36

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

WORKDIR $HOME

# This are libraries needed for napari
RUN apt-get update -y && \
    apt-get install -y --no-install-recommends \
    build-essential \
    curl  \
    libdbus-1-3 \
    libfontconfig1 \
    libgl1-mesa-glx \
    libglib2.0-0 \
    libxcb-icccm4 \
    libxcb-image0 \
    libxcb-keysyms1 \
    libxcb-randr0 \
    libxcb-render-util0 \
    libxcb-shape0  \
    libxcb-xfixes0 \
    libxcb-xinerama0 \
    libxcb-xinput0 \
    libxi6 \
    libxkbcommon-x11-0 \
    libxrender1 \
    mesa-utils &&\
    rm -rf /var/lib/apt/lists

# v5.0.1
RUN git clone https://github.com/neuropoly/axondeepseg.git && \
    cd axondeepseg && \
    git checkout ccd48b1e9d4738c9f30fe5a1a93e4f283460a3b8 &&\
    ./install_ads -y

WORKDIR /home/scu/work

ENV DISPLAY=:0
ENV XPRA_PORT=9876
EXPOSE 9876


# -----------------------------------
COPY config/menu.xml /etc/xdg/openbox/
COPY config/supervisord.conf /etc/

# Overwrite openbox config to start full screen
COPY config/rc.xml /etc/xdg/openbox/rc.xml
## working directory and permissions

COPY docker/ /docker

CMD ["sh", "-c", "/docker/boot.sh"]


# source /root/.bashrc
# ads_napari