FROM rootproject/root:6.22.02-ubuntu20.04
WORKDIR /soft

ARG JOBS=4

# Build Pythia8
ARG PYTHIA_VER=8235
ENV PYTHIA_URL=http://home.thep.lu.se/~torbjorn/pythia8/pythia${PYTHIA_VER}.tgz
ENV PYTHIA_DIR pythia${PYTHIA_VER}
ENV PYTHIA /soft/${PYTHIA_DIR}

RUN wget -q ${PYTHIA_URL} && \
    tar -xzf ${PYTHIA_DIR}.tgz && \
    rm -f ${PYTHIA_DIR}.tgz && \ 
    cd ${PYTHIA_DIR} && \
    ./configure && \
    make -j${JOBS}

# Build FastJet
ARG FASTJET_VER=3.3.3
ENV FASTJET_URL http://fastjet.fr/repo/fastjet-${FASTJET_VER}.tar.gz
ARG FASTJET_DIR=fastjet-${FASTJET_VER}
ENV FASTJET /soft/${FASTJET_DIR}-install

RUN wget -q ${FASTJET_URL} && \
    tar -xzf ${FASTJET_DIR}.tar.gz && \
    rm -f ${FASTJET_DIR}.tar.gz && \
    cd ${FASTJET_DIR} && \
    mkdir ../${FASTJET_DIR}-install && \
    ./configure --prefix=$PWD/../${FASTJET_DIR}-install &&\
    make -j${JOBS} && make check && make install

# Build FastJetContrib
ARG FJ_CONTRIB_VER=1.042
ENV FJ_CONTRIB_URL http://fastjet.hepforge.org/contrib/downloads/fjcontrib-${FJ_CONTRIB_VER}.tar.gz 
ENV FJ_CONTRIB_DIR fjcontrib-${FJ_CONTRIB_VER}

RUN wget -q ${FJ_CONTRIB_URL} && \
    tar -xzf ${FJ_CONTRIB_DIR}.tar.gz && \
    rm -f ${FJ_CONTRIB_DIR}.tar.gz && \
    cd fjcontrib-"$FJ_CONTRIB_VER" &&\
    ./configure --fastjet-config=${FASTJET}/bin/fastjet-config --prefix=`${FASTJET}/bin/fastjet-config --prefix` && \
    make -j${JOBS} && \ 
    make install && \ 
    make fragile-shared && \
    make fragile-shared-install

# Build JetToyHI
ARG JET_TOY_HI_REPO=https://github.com/mverwe/JetToyHI.git

RUN apt-get update && \
    apt-get install -y git \
    xutils-dev


RUN git clone ${JET_TOY_HI_REPO}&& \
    cd JetToyHI && \
    git pull --rebase origin forbsc2 && \
    echo `${FASTJET}/bin/fastjet-config --prefix` > .fastjet && \
    echo ${PYTHIA} > .pythia8

# Build PU14
RUN cd JetToyHI/PU14 && \
    echo `${FASTJET}/bin/fastjet-config --prefix` > .fastjet && \
    ./mkmk && \
    make -j${JOBS}

RUN cd JetToyHI && \
    scripts/mkcxx.pl -f -s -1 -r -8 '-IPU14' -l '-LPU14 -lPU14 -lz' && \
    make -j${JOBS}

WORKDIR /user_home
ENTRYPOINT [ "/bin/bash" ]