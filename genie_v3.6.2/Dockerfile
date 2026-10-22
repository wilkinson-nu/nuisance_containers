FROM wilkinsonnu/nuisancemc:root_v6.30.08

ARG NCORES=4
ENV GEN_DIR=/opt/generators

# Install build deps
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    python-is-python3 liblog4cpp5-dev && \
    rm -rf /var/lib/apt/lists/*

## Sort out LHAPDF
WORKDIR ${GEN_DIR}/LHAPDF
ARG LHAPDF_VERSION=6.5.6
RUN wget https://lhapdf.hepforge.org/downloads/LHAPDF-${LHAPDF_VERSION}.tar.gz && \
    tar -xzf LHAPDF-${LHAPDF_VERSION}.tar.gz && \
    rm LHAPDF-${LHAPDF_VERSION}.tar.gz && \
    cd LHAPDF-${LHAPDF_VERSION} && \
    ./configure --prefix=/opt/generators/LHAPDF-${LHAPDF_VERSION}_build && \
    make -j${NCORES} && \
    make install

ENV LHAPDF=${GEN_DIR}/LHAPDF-${LHAPDF_VERSION}_build
ENV LD_LIBRARY_PATH=${LHAPDF}/lib:${LD_LIBRARY_PATH}
ENV PATH=${LHAPDF}/bin:${PATH}
ENV LHAPATH=${LHAPDF}/share/LHAPDF

## Setup the GENIE paths
ENV GENIE_VERSION=R-3_06_02
ENV GENIE=${GEN_DIR}/GENIE/${GENIE_VERSION}
ENV LHAPDF_LIB=${LHAPDF}/lib
ENV LHAPDF_INC=${LHAPDF}/include

## Get the GENIE code
WORKDIR ${GEN_DIR}/GENIE
RUN git clone -b ${GENIE_VERSION} --depth 1 --single-branch https://github.com/GENIE-MC/Generator.git ${GENIE} && \    
    cd ${GENIE} && \
    sed -i 's/g77/gfortran/g' src/make/Make.include && \
    ./configure --enable-debug \
    		--enable-fnal \
    		--enable-rwght \
    		--enable-lhapdf6 \
		--disable-lhapdf5 \
    		--disable-apfel && \
    make -j${NCORES} && make install
    
ENV PATH=$GENIE/bin:${PATH}
ENV LD_LIBRARY_PATH=$GENIE/lib:$LD_LIBRARY_PATH

## Use a custom Messenger to quieten things down:
COPY Messenger.xml ${GENIE}/config/Messenger.xml

## Get the GENIERW code
ENV GENIERW_VERSION=R-1_04_02
ENV GENIE_REWEIGHT=${GEN_DIR}/GENIERW/${GENIERW_VERSION}
RUN git clone -b ${GENIERW_VERSION} --depth 1 --single-branch https://github.com/GENIE-MC/Reweight.git ${GENIE_REWEIGHT} && \
    cd ${GENIE_REWEIGHT} && \
    make -j${NCORES} && make install

## Get and build NUISANCE (using a fixed commit for reproducibility)
WORKDIR ${GEN_DIR}
RUN git clone https://github.com/NUISANCEMC/nuisance.git NUISANCE && \
    cd NUISANCE && git checkout cc78ce0 && \
    mkdir build && cd build && \
    cmake -DCMAKE_CXX_STANDARD=17 \
          -DGENIE_ENABLED=1 \
	  -DNEUT_ENABLED=0 \
          -DProb3plusplus_ENABLED=0 \
          -Dnusystematics_ENABLED=0 \
          ../ && \
    make -j${NCORES} && make install

ENV NUISANCE=${GEN_DIR}/NUISANCE
ENV PATH=${NUISANCE}/build/Linux/bin:${PATH}
ENV LD_LIBRARY_PATH=${NUISANCE}/build/Linux/lib:${LD_LIBRARY_PATH}
