FROM wilkinsonnu/nuisancemc:root_v6.30.08

ARG NCORES=4
ENV GEN_DIR=/opt/generators

# Install build deps
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    libxm4 libxt-dev libxaw7-dev libmotif-dev \
    unzip autoconf libtool automake && \
    rm -rf /var/lib/apt/lists/*

## Sort out CERNLIB (the builtin version is currently not available due to CERN taking the tarball behind a password)
COPY cernlib-2023.10.31.0-free.tar.gz /tmp/.
ARG CERNLIB_VERSION=2023.10.31.0-free
RUN cd /tmp && \
    tar -xzf cernlib-${CERNLIB_VERSION}.tar.gz && \
    cd cernlib-${CERNLIB_VERSION} && \
    sed -i 's/CMAKE_VERBOSE_MAKEFILE ON/CMAKE_VERBOSE_MAKEFILE OFF/g' CMakeLists.txt && \
    cd paw_motif && \
    ln -s xbae Xbae

## BUILD CERNLIB
RUN mkdir -p /tmp/cernlib-build && \
    cd /tmp/cernlib-build && \
    cmake /tmp/cernlib-${CERNLIB_VERSION}/ \
        -DCMAKE_INSTALL_PREFIX=${GEN_DIR}/CERNLIB \
        -DCMAKE_INSTALL_LIBDIR=lib \
        -DCERNLIB_POSITION_INDEPENDENT_CODE=ON \
        -DCERNLIB_BUILD_SHARED=OFF \
        -DCMAKE_BUILD_TYPE=RELEASE && \
    make -j${NCORES} && \
    make install && \
    rm -rf /tmp/cernlib-build /tmp/cernlib-${CERNLIB_VERSION} \
           /tmp/cernlib-${CERNLIB_VERSION}.tar.gz

## Variables NEUT expects to find CERNLIB
ENV CERN=${GEN_DIR}/CERNLIB
ENV CERN_LEVEL=2023

## Copy in the NEUT tarball
ARG NEUT_VERSION=5.9.2
ARG NUCDEEX_VERSION=v2.1.2
COPY neut-${NEUT_VERSION}.zip ${GEN_DIR}/neut-${NEUT_VERSION}.zip

## Unpack and build NEUT (also get the NucDeEx code)
WORKDIR ${GEN_DIR}
RUN unzip neut-${NEUT_VERSION}.zip && \
    mv neut-${NEUT_VERSION} neut && \
    rm neut-${NEUT_VERSION}.zip && \
    git clone -b ${NUCDEEX_VERSION} --depth 1 https://github.com/SeishoAbe/NucDeEx.git ${GEN_DIR}/neut/src/nucdeex/nucdeex && \
    cd neut && \
    cd src && autoreconf -i --force && cd .. && \
    mkdir build && cd build && \
    ../src/configure --prefix=${GEN_DIR}/neut/build/Linux && \
    make -j${NCORES} && make install

ENV NEUT_ROOT=${GEN_DIR}/neut
ENV NEUT=${GEN_DIR}/neut/build/Linux
ENV NEUT_CARDS=${NEUT}/share/neut/Cards
ENV NEUT_CRSPATH=${NEUT}/share/neut/crsdat
ENV PATH=${NEUT}/bin:${PATH}
ENV LD_LIBRARY_PATH=${NEUT}/lib:${LD_LIBRARY_PATH}
ENV PKG_CONFIG_PATH=${NEUT}

## Get and build NUISANCE (using a fixed commit for reproducibility)
WORKDIR ${GEN_DIR}
RUN git clone https://github.com/NUISANCEMC/nuisance.git NUISANCE && \
    cd NUISANCE && git checkout cc78ce0 && \
    mkdir build && cd build && \
    cmake -DCMAKE_CXX_STANDARD=17 \
          -DNEUT_ENABLED=1 \
          -DProb3plusplus_ENABLED=0 \
          -Dnusystematics_ENABLED=0 \
          ../ && \
    make -j${NCORES} && make install

ENV NUISANCE=${GEN_DIR}/NUISANCE
ENV PATH=${NUISANCE}/build/Linux/bin:${PATH}
ENV LD_LIBRARY_PATH=${NUISANCE}/build/Linux/lib:${LD_LIBRARY_PATH}
