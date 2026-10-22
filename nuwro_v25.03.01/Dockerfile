FROM wilkinsonnu/nuisancemc:root_v6.30.08

ARG NCORES=4
ENV GEN_DIR=/opt/generators

# Install build deps
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        libxml2-utils && \
    rm -rf /var/lib/apt/lists/*

## Symlink Pythia6 into ROOT lib dir as NuWro expects
RUN ln -s ${GEN_DIR}/pythia6/libPythia6.so ${ROOTSYS}/lib/libPythia6.so

## Get NuWro (using a slightly modified version to avoid a compilation error... a PR has been submitted...)
WORKDIR ${GEN_DIR}
RUN git clone --depth 1 --branch remove-vla https://github.com/wilkinson-nu/nuwro nuwro && \
    cd nuwro && make -j${NCORES}

ENV NUWRO=${GEN_DIR}/nuwro
ENV PATH=${NUWRO}/bin:${PATH}
ENV PYTHIA6=${GEN_DIR}/pythia6
ENV ROOT_INCLUDE_PATH=${NUWRO}/src

## Get and build NUISANCE (using a fixed commit for reproducibility)
WORKDIR ${GEN_DIR}
RUN git clone https://github.com/NUISANCEMC/nuisance.git NUISANCE && \
    cd NUISANCE && git checkout cc78ce0 && \
    mkdir build && cd build && \
    cmake -DCMAKE_CXX_STANDARD=17 \
          -DNuWro_ENABLED=1 \
          -DREWEIGHT_ENABLED=0 \
          -DHepMC3_ENABLED=0 \
          -DProb3plusplus_ENABLED=0 \
          -Dnusystematics_ENABLED=0 \
          ../ && \
    make -j${NCORES} && make install

ENV NUISANCE=${GEN_DIR}/NUISANCE
ENV PATH=${NUISANCE}/build/Linux/bin:${PATH}
ENV LD_LIBRARY_PATH=${NUISANCE}/build/Linux/lib:${LD_LIBRARY_PATH}

