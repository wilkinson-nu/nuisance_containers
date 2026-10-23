FROM wilkinsonnu/nuisancemc:root_v6.30.08

ARG NCORES=4
ENV GEN_DIR=/opt/generators

## Install build deps
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    libbz2-dev && \
    rm -rf /var/lib/apt/lists/*

## Get and build GiBUU
ENV GiBUU=${GEN_DIR}/GiBUU
RUN mkdir ${GiBUU} && cd ${GiBUU} && \
    wget --content-disposition https://gibuu.hepforge.org/downloads?f=archive/r2025_05/release2025.tar.gz && \
    tar -xzf release2025.tar.gz && rm release2025.tar.gz && \
    wget --content-disposition https://gibuu.hepforge.org/downloads?f=archive/r2025_05/buuinput2025.tar.gz && \
    tar -xzf buuinput2025.tar.gz && rm buuinput2025.tar.gz && \
    wget --content-disposition https://gibuu.hepforge.org/downloads?f=archive/r2025_05/libraries2025_RootTuple.tar.gz && \
    tar -xzf libraries2025_RootTuple.tar.gz && rm libraries2025_RootTuple.tar.gz && \
    cd release2025 && \
    make buildRootTuple_POS && \
    make withROOT=1 -j${NCORES}
    
RUN ln -s ${GiBUU}/release2025/testRun/GiBUU.x ${GiBUU}/.

ENV PATH=/opt/generators/GiBUU/:${PATH}
    
## Get and build NUISANCE (using a fixed commit for reproducibility)
WORKDIR ${GEN_DIR}
RUN git clone https://github.com/NUISANCEMC/nuisance.git NUISANCE && \
    cd NUISANCE && git checkout cc78ce0 && \
    mkdir build && cd build && \
    cmake -DCMAKE_CXX_STANDARD=17 \
          -DGiBUU_ENABLED=1 \
	  -DNEUT_ENABLED=0 \
          -DProb3plusplus_ENABLED=0 \
          -Dnusystematics_ENABLED=0 \
          ../ && \
    make -j${NCORES} && make install

ENV NUISANCE=${GEN_DIR}/NUISANCE
ENV PATH=${NUISANCE}/build/Linux/bin:${PATH}
ENV LD_LIBRARY_PATH=${NUISANCE}/build/Linux/lib:${LD_LIBRARY_PATH}
