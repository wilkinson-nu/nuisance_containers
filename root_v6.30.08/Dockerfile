FROM debian:12-slim

ARG NCORES=4
ENV GEN_DIR=/opt/generators

# Install build deps
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential gfortran cmake git wget \
    libboost-all-dev libclhep-dev \
    python3 python3-pip \
    python3-numpy python3-dev \
    zlib1g-dev libx11-dev libssl-dev libpng-dev libjpeg-dev \
    liblzma-dev libxxhash-dev libxpm-dev libxft-dev libxext-dev \
    libgsl-dev liblz4-dev libgif-dev libtiff-dev libxml2-dev \
    libfftw3-dev libblas-dev ca-certificates \
    libxerces-c-dev libhdf5-dev \
    libglew-dev libyaml-cpp-dev \
    nlohmann-json3-dev libzstd-dev \
    libpcre3-dev libfreetype-dev \
    && update-ca-certificates

# Build dir
WORKDIR ${GEN_DIR}

# Get PYTHIA6 (old version before CERN took these behind a password...
#RUN wget --no-check-certificate http://root.cern.ch/download/pythia6.tar.gz && \
#    tar -xzf pythia6.tar.gz && rm pythia6.tar.gz && \
#    wget --no-check-certificate https://pythia.org/download/pythia6/pythia6428.f && \
#    mv pythia6428.f pythia6/pythia6428.f && rm pythia6/pythia6416.f

## Replacement from local files
COPY pythia6.tar.gz ${GEN_DIR}/pythia6.tar.gz
COPY pythia6428.f ${GEN_DIR}/pythia6428.f
RUN tar -xzf pythia6.tar.gz && rm pythia6.tar.gz && \
    mv pythia6428.f pythia6/pythia6428.f && rm pythia6/pythia6416.f	

# Build PYTHIA6
WORKDIR ${GEN_DIR}/pythia6
RUN sed -i "s/int /extern int /g" pythia6_common_address.c && \
    sed -i "s/char /extern char /g" pythia6_common_address.c && \
    sed -i "s/extern int pyuppr/int pyuppr/g" pythia6_common_address.c && \
    sed -i "s/-m64//g" makePythia6.linuxx8664 && \
    ./makePythia6.linuxx8664

ENV PYTHIA6=${GEN_DIR}/pythia6
ENV LD_LIBRARY_PATH=${PYTHIA6}:${LD_LIBRARY_PATH}

# Get and build ROOT
RUN git clone -b v6-30-08 --depth 1 --single-branch https://github.com/root-project/root.git /tmp/root-src && \
    mkdir /tmp/root-build && \
    cd /tmp/root-build && \
    cmake -DCMAKE_INSTALL_PREFIX=${GEN_DIR}/root \
    	  -DPYTHIA6_LIBRARY=${PYTHIA6}/libPythia6.so \
          -DCMAKE_CXX_STANDARD=17 \
	  -Dpythia6=ON -Dminuit2=ON -Dmathmore=ON -Dgdml=ON \
          -Ddavix=OFF -Dfitsio=OFF -Dgfal=OFF -Dcastor=OFF \
          -Dclad=OFF -Dhttp=OFF -Droot7=OFF -Dwebgui=OFF \
          -Dxrootd=OFF -Dmlp=OFF -Dmysql=OFF -Doracle=OFF \
          -Dpgsql=OFF -Droofit=OFF -Dspectrum=OFF -Dsqlite=OFF \
          -Ddataframe=ON -Dimt=OFF -Dtmva=OFF -Dvdt=OFF \
          -Dtmva-cpu=OFF -Dtmva-pymva=OFF -Dssl=OFF -Dcudnn=OFF \
	  -Deve=OFF -Dx11=ON -Dopengl=OFF \
          -DPYTHON_EXECUTABLE=/usr/bin/python3 \
	  -Druntime_cxxmodules=OFF \
	  -Dbuiltin_afterimage=ON \
	  -Dfail-on-missing=ON \
	  /tmp/root-src && \
    make -j${NCORES} && make install && \
    rm -rf /tmp/root-src /tmp/root-build

## Set ROOT environment
ENV ROOTSYS=${GEN_DIR}/root
ENV PATH=${ROOTSYS}/bin:${PATH}
ENV LD_LIBRARY_PATH=${ROOTSYS}/lib:${LD_LIBRARY_PATH}
