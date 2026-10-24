## Sources
FROM wilkinsonnu/nuisancemc:genie_v3.6.2 AS genie
FROM wilkinsonnu/nuisancemc:gibuu_v2025p5 AS gibuu
FROM wilkinsonnu/nuisancemc:neut_v5.9.2 AS neut
FROM wilkinsonnu/nuisancemc:nuwro_v25.03.01 AS nuwro

## Final image base
FROM wilkinsonnu/nuisancemc:root_v6.30.08

ARG NCORES=4

## Add runtime libraries needed by generators
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        liblog4cpp5-dev && \
    rm -rf /var/lib/apt/lists/*

WORKDIR ${GEN_DIR}

## GENIE + REWEIGHT + LHAPDF 
ENV GENIE=${GEN_DIR}/GENIE/R-3_06_02
ENV LHAPDF=${GEN_DIR}/LHAPDF-6.5.6_build
ENV GENIE_REWEIGHT=${GEN_DIR}/GENIERW/R-1_04_02

COPY --from=genie ${LHAPDF} ${LHAPDF}
COPY --from=genie ${GENIE} ${GENIE}
COPY --from=genie ${GENIE_REWEIGHT} ${GENIE_REWEIGHT}

ENV LHAPATH=${LHAPDF}/share/LHAPDF
ENV LHAPDF_INC=${LHAPDF}/include 
ENV LHAPDF_LIB=${LHAPDF}/lib 
ENV LD_LIBRARY_PATH=${GENIE_REWEIGHT}/lib:${GENIE}/lib:${LHAPDF}/lib:${LD_LIBRARY_PATH}
ENV PATH=${GENIE_REWEIGHT}/bin:${GENIE}/bin:${LHAPDF}/bin:${PATH}

## NEUT
ENV NEUT=${GEN_DIR}/neut/build/Linux
ENV CERN=${GEN_DIR}/CERNLIB

COPY --from=neut ${CERN}/lib ${CERN}/lib
COPY --from=neut ${NEUT}/bin ${NEUT}/bin
COPY --from=neut ${NEUT}/lib ${NEUT}/lib
COPY --from=neut ${NEUT}/share ${NEUT}/share
COPY --from=neut ${NEUT}/include ${NEUT}/include
COPY --from=neut ${NEUT}/cmake ${NEUT}/cmake
COPY --from=neut ${NEUT}/NEUT.pc ${NEUT}/NEUT.pc

ENV NEUT_ROOT=${GEN_DIR}/neut
ENV NEUT=${GEN_DIR}/neut/build/Linux
ENV NEUT_CARDS=${NEUT}/share/neut/Cards
ENV NEUT_CRSPATH=${NEUT}/share/neut/crsdat
ENV PATH=${NEUT}/bin:${PATH}
ENV LD_LIBRARY_PATH=${NEUT}/lib:${LD_LIBRARY_PATH}
ENV PKG_CONFIG_PATH=${NEUT}

## NuWro
ENV NUWRO=${GEN_DIR}/nuwro

COPY --from=nuwro ${NUWRO}/bin ${NUWRO}/bin
COPY --from=nuwro ${NUWRO}/data ${NUWRO}/data
COPY --from=nuwro ${NUWRO}/src ${NUWRO}/src

ENV PATH=${NUWRO}/bin:${PATH}
ENV ROOT_INCLUDE_PATH=${NUWRO}/src

## GiBUU
ENV GiBUU=${GEN_DIR}/GiBUU

COPY --from=gibuu ${GiBUU}/release2025 ${GiBUU}/release2025
COPY --from=gibuu ${GiBUU}/buuinput2025 ${GiBUU}/buuinput2025
COPY --from=gibuu ${GiBUU}/libraries2025 ${GiBUU}/libraries2025

RUN ln -s ${GiBUU}/release2025 ${GiBUU}/release && \
    ln -s ${GiBUU}/buuinput2025 ${GiBUU}/buuinput && \
    ln -s ${GiBUU}/libraries2025 ${GiBUU}/libraries && \
    ln -s ${GiBUU}/release2025/testRun/GiBUU.x ${GiBUU}/GiBUU.x

ENV PATH=${GiBUU}:${PATH}

## Build NUISANCE
RUN git clone https://github.com/NUISANCEMC/nuisance.git NUISANCE && \
    cd NUISANCE && \
    git checkout cc78ce0 && \
    mkdir build && \
    cd build && \
    cmake -DCMAKE_CXX_STANDARD=17 \
          -DGENIE_ENABLED=1 \
          -DGiBUU_ENABLED=1 \
          -DNEUT_ENABLED=1 \
          -DNuWro_ENABLED=1 \
          -DREWEIGHT_ENABLED=0 \
          -DHepMC3_ENABLED=0 \
          -DProb3plusplus_ENABLED=0 \
          -Dnusystematics_ENABLED=0 \
          ../ && \
    make -j${NCORES} && \
    make install

ENV NUISANCE=${GEN_DIR}/NUISANCE
ENV PATH=${NUISANCE}/build/Linux/bin:${PATH}
ENV LD_LIBRARY_PATH=${NUISANCE}/build/Linux/lib:${LD_LIBRARY_PATH}