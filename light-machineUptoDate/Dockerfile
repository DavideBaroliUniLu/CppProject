# Builds a base Docker image that all other cppproject images are
# built off of (FROM). In this image we control the end user
# experience, namely; creating the user lifeV, setting up the users
# environment and the default entry point and cmds.
#
# Although this image is not used directly by end-users, it can be
# found at:
#
#   cpproject/LifeVLib
#
# Authors:
# Davide Baroli <davide.baroli@uni.lu>

FROM phusion/baseimage:0.9.19

# Get Ubuntu updates
USER root
RUN apt-get update -q && \ 
    apt-get upgrade -y -o Dpkg::Options::="--force-confold" && \
    apt-get -y install sudo && \
    apt-get -y install locales && \
    echo "C.UTF-8 UTF-8" > /etc/locale.gen && \
    locale-gen && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# Set locale environment
ENV LC_ALL=C.UTF-8 \
    LANG=C.UTF-8 \
    LANGUAGE=C.UTF-8

# OpenBLAS threads should be 1 to ensure performance
RUN echo 1 > /etc/container_environment/OPENBLAS_NUM_THREADS && \
    echo 0 > /etc/container_environment/OPENBLAS_VERBOSE

# Set up user so that we do not run as root
RUN useradd -m -s /bin/bash -G sudo,docker_env PyGeM && \
    echo "CppDev:docker" | chpasswd && \
    echo "CppDev ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers

RUN touch /etc/service/syslog-forwarder/down
COPY set-home-permissions.sh /etc/my_init.d/set-home-permissions.sh
RUN chmod +x /etc/my_init.d/set-home-permissions.sh

USER CppDev
ENV HOME /home/CppDev
RUN touch $HOME/.sudo_as_admin_successful && \
    mkdir $HOME/shared && \
    mkdir $HOME/build && \
    mkdir $HOME/install

VOLUME /home/CppDev/shared

WORKDIR /home/CppDev
ENTRYPOINT ["sudo","/sbin/my_init","--quiet","--","sudo","-u","CppDev","/bin/bash","-l","-c"]
CMD ["/bin/bash","-i"]

# utilities and libraries
USER root
RUN apt-get update -y; apt-get install -y --force-yes --fix-missing --no-install-recommends curl git unzip tree subversion vim cmake bison g++ gfortran openmpi-bin pkg-config wget libpcre3-dev bison flex swig libglu1-mesa pyqt4-dev-tools gcc git libblas-dev \
liblapack-dev libopenmpi-dev ninja-build \
numdiff openmpi-bin python bison flex ca-certificates \
bzip2 tar zlib1g-dev vim libgmp-dev libmpfr-dev libbz2-dev libcppunit-dev libvtk6-dev  libhwloc5 libbison-dev flex
RUN apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

RUN id CppDev
RUN chown -R CppDev:CppDev $HOME
# Astyle
RUN cd /tmp && \
    wget http://sourceforge.net/projects/astyle/files/astyle/astyle%202.04/astyle_2.04_linux.tar.gz && \
    tar xf astyle_2.04_linux.tar.gz && \
    cd astyle/build/gcc && \
    make && \
    USER=root make install && \
    rm /tmp/*

# Environment Setting
ENV CMAKE_VERSION=3.2.2 \
    HDF5_VERSION=1.8.14 \
    PETSC_VERSION=3.7.0 \
    SLEPC_VERSION=3.7.0 \
    TRILINOS_VERSION=12.6.4 

ENV TRILINOS_VER 12-6-4

WORKDIR /tmp


RUN wget http://www.cmake.org/files/v3.2/cmake-${CMAKE_VERSION}.tar.gz && \
    tar xzvf cmake-${CMAKE_VERSION}.tar.gz && \
    cd cmake-${CMAKE_VERSION} && \
    ./configure --prefix=/usr/local && \
    make && make install && \
    rm -rf /tmp/*


RUN export NO_BZIP2=1
RUN wget --read-timeout=10 -nc http://sourceforge.net/projects/boost/files/boost/1.60.0/boost_1_60_0.tar.gz/download -O boost.tar.gz && \
    tar -xf boost.tar.gz && \
    cd boost_1_60_0 && \
    touch user-config.jam && \
     echo "using mpi ;" > user-config.jam && \
     ./bootstrap.sh --prefix=$ENVDEV_HOME --show-libraries  --with-libraries=mpi,filesystem,program_options,system,thread,timer,iostreams,python 
RUN  cd boost_1_60_0 && \
      echo "the folder is " $PWD && \
      ./b2 --prefix=/usr/local  -s  NO_BZIP2=1  -d2 -j2  \
       link=shared threading=multi runtime-link=shared --layout=tagged --user-config=user-config.jam  && \
     ./b2 install  
RUN    rm -rf /tmp/*

RUN wget https://www.hdfgroup.org/ftp/HDF5/releases/hdf5-${HDF5_VERSION}/src/hdf5-${HDF5_VERSION}.tar.bz2 &&\
    tar xjvf hdf5-${HDF5_VERSION}.tar.bz2 && \
    cd hdf5-${HDF5_VERSION}&& \
    ./configure --enable-parallel --enable-shared --prefix=/usr/local && \
    make && \
    make install&& \
    rm -rf /tmp/*


# Install PETSc from source
RUN wget --quiet -nc http://ftp.mcs.anl.gov/pub/petsc/release-snapshots/petsc-lite-${PETSC_VERSION}.tar.gz && \
    tar -xf petsc-lite-${PETSC_VERSION}.tar.gz && \
    cd petsc-${PETSC_VERSION} && \
    ./configure --COPTFLAGS="-O2" \
                --CXXOPTFLAGS="-O2" \
                --FOPTFLAGS="-O2" \
                --with-blas-lib=/usr/lib/libblas.so --with-lapack-lib=/usr/lib/liblapack.so \
                --with-c-support \
                --with-debugging=0 \
                --with-shared-libraries \
                --download-suitesparse \
                --download-scalapack \
                --download-metis \
                --download-parmetis \
                --download-ptscotch \
                --download-hypre \
                --download-mumps \
                --download-blacs \
                --download-spai \
                --download-ml \
                --prefix=/usr/local && \
     make && \
     make install && \
     rm -rf /tmp/*

# Install SLEPc from source
RUN export PETSC_DIR=/usr/local && \
    wget -nc --quiet http://www.grycap.upv.es/slepc/download/download.php?filename=slepc-${SLEPC_VERSION}.tar.gz -O slepc-${SLEPC_VERSION}.tar.gz && \
    tar -xf slepc-${SLEPC_VERSION}.tar.gz && \
    cd slepc-${SLEPC_VERSION} && \
    ./configure --prefix=/usr/local && \
    make && \
    make install && \
    rm -rf /tmp/*
ENV SLEPC_DIR=/usr/local \
    PETSC_DIR=/usr/local \
    BOOST_DIR=/usr/local \
    HDF5_DIR=/usr/local 



RUN cd /tmp && \
    wget -nc https://github.com/trilinos/Trilinos/archive/trilinos-release-${TRILINOS_VER}.tar.gz && \ 
    mv trilinos-release-${TRILINOS_VER}.tar.gz  trilinos-${TRILINOS_VERSION}-Source.tar.gz && \ 
    tar -zxvf trilinos-${TRILINOS_VERSION}-Source.tar.gz && \ 
    mv Trilinos-trilinos-release-${TRILINOS_VER} trilinos-${TRILINOS_VERSION}-Source

RUN cd /tmp && \
    cd trilinos-${TRILINOS_VERSION}-Source && \
    mkdir build && cd build && \
    cmake -DCMAKE_INSTALL_PREFIX=/usr/local \
          -D TPL_ENABLE_MPI:BOOL=ON \
          -D MPI_CXX_COMPILER:FILEPATH=mpicxx \
          -D MPI_C_COMPILER:FILEPATH=mpicc \
          -D CMAKE_CXX_FLAGS="-O2" \
          -D CMAKE_C_FLAGS="-O2" \
          -D Trilinos_ENABLE_CXX11=ON \
          -D Trilinos_ENABLE_ALL_PACKAGES:BOOL=OFF \
          -D BUILD_SHARED_LIBS:BOOL=ON \
          -D CMAKE_BUILD_TYPE:STRING=Release \
          -D TPL_ENABLE_HDF5:BOOL=ON \
          -D HDF5_INCLUDE_DIRS:PATH=${HDF5_DIR}/include \
          -D HDF5_LIBRARY_DIRS:PATH=${HDF5_DIR}/lib \
          -D TPL_ENABLE_Scotch:BOOL=ON \
          -D TPL_ENABLE_Boost:BOOL=ON \
          -D TPL_ENABLE_ParMETIS:BOOL=ON \
          -D Boost_INCLUDE_DIRS:PATH=${BOOST_DIR}/include \
          -D Boost_LIBRARY_DIRS:PATH=${BOOST_DIR}/lib \
          -D TPL_ENABLE_BoostLib:BOOL=ON \
          -D BoostLib_INCLUDE_DIRS:PATH=${BOOST_DIR}/include \
          -D BoostLib_LIBRARY_DIRS:PATH=${BOOST_DIR}/lib \
          -D TPL_ENABLE_Matio=OFF \
          -D TPL_ENABLE_GLM:BOOL=OFF \
          -D TPL_ENABLE_BLAS:BOOL=ON \
          -D Boost_NO_BOOST_CMAKE:BOOL=ON \
          -D Trilinos_ENABLE_EXPLICIT_INSTANTIATION:BOOL=ON \
          -D Trilinos_ENABLE_TESTS:BOOL=OFF \
          -D Trilinos_ENABLE_EXAMPLES:BOOL=OFF \
          -D Trilinos_ASSERT_MISSING_PACKAGES:BOOL=OFF \
    	    -D Trilinos_ENABLE_ALL_OPTIONAL_PACKAGES:BOOL=OFF \
          -D Trilinos_ENABLE_Gtest:BOOL=ON \
          -D Trilinos_ENABLE_ThreadPool:BOOL=ON \
          -D Trilinos_ENABLE_KokkosCore:BOOL=ON \
          -D Trilinos_ENABLE_KokkosContainers:BOOL=ON \
          -D Trilinos_ENABLE_KokkosAlgorithms:BOOL=ON \
          -D Trilinos_ENABLE_Kokkos:BOOL=ON \
          -D Trilinos_ENABLE_TeuchosCore:BOOL=ON \
          -D Trilinos_ENABLE_TeuchosParameterList:BOOL=ON \
          -D Trilinos_ENABLE_TeuchosComm:BOOL=ON \
          -D Trilinos_ENABLE_TeuchosNumerics:BOOL=ON \
          -D Trilinos_ENABLE_TeuchosRemainder:BOOL=ON \
          -D Trilinos_ENABLE_TeuchosKokkosCompat:BOOL=ON \
          -D Trilinos_ENABLE_TeuchosKokkosComm:BOOL=ON \
          -D Trilinos_ENABLE_Teuchos:BOOL=ON \
          -D Trilinos_ENABLE_RTOp:BOOL=ON \
          -D Trilinos_ENABLE_Sacado:BOOL=ON \
          -D Trilinos_ENABLE_Rol:BOOL=ON \
          -D Trilinos_ENABLE_NOX:BOOL=ON \
          -D Trilinos_ENABLE_Epetra:BOOL=ON \
          -D Trilinos_ENABLE_TpetraClassic:BOOL=ON \
          -D Trilinos_ENABLE_TpetraKernels:BOOL=ON \
          -D Trilinos_ENABLE_TpetraCore:BOOL=ON \
          -D Trilinos_ENABLE_Tpetra:BOOL=ON \
          -D Trilinos_ENABLE_Xpetra:BOOL=ON \
          -D Trilinos_ENABLE_Belos:BOOL=ON \
          -D Trilinos_ENABLE_Ifpack:BOOL=ON \
          -D Trilinos_ENABLE_Amesos:BOOL=ON \
          -D Trilinos_ENABLE_Ifpack2:BOOL=ON \
          -D Trilinos_ENABLE_Amesos2:BOOL=ON \
          -D Trilinos_ENABLE_PyTrilinos:BOOL=OFF\
          -D Trilinos_ENABLE_ShyLU:BOOL=ON \
          -D Trilinos_ENABLE_Teko:BOOL=ON \
          -D IFPACK_PARALLEL_SUBDOMAIN_SOLVERS:BOOL=ON\
          -D IFPACK_DYNAMIC_FACTORY:BOOL=ON\
          -D Trilinos_ENABLE_MueLu:BOOL=ON \
          -D Trilinos_ENABLE_Amesos2:BOOL=ON \
          -D Amesos2_ENABLE_KLU2:BOOL=ON \ 
          -D Amesos2_ENABLE_EXPLICIT_INSTANTIATION:BOOL=ON \
          -D Ifpack2_ENABLE_EXPLICIT_INSTANTIATION:BOOL=ON \
          -D MueLu_ENABLE_EXPLICIT_INSTANTIATION:BOOL=ON \
          -D TpetraCore_ENABLE_EXPLICIT_INSTANTIATION:BOOL=ON \
          -D Tpetra_ENABLE_EXPLICIT_INSTANTIATION:BOOL=ON \
          -D Tpetra_INST_COMPLEX_DOUBLE:BOOL=ON \
          -D Tpetra_INST_DOUBLE:BOOL=ON \
          -D Tpetra_INST_INT_INT:BOOL=ON \
          -D Tpetra_INST_INT_LONG_LONG:BOOL=ON \
          -D Tpetra_INST_SERIAL:BOOL=ON \
          -D TpetraCore_ENABLE_EXPLICIT_INSTANTIATION:BOOL=ON \
          -D Xpetra_ENABLE_EXPLICIT_INSTANTIATION:BOOL=ON \
          -D Trilinos_NO_64BIT_GLOBAL_INDICES:BOOL=ON \
          -D Trilinos_NO_32BIT_GLOBAL_INDICES:BOOL=OFF \
          -D Teuchos_ENABLE_LONG_LONG_INT:BOOL=ON \
          .. && \
    make  && \
    make install && \
    rm -rf /tmp/*

USER CppDev 

