FROM  ubuntu:24.04

RUN apt-get update
RUN apt-get install -y sudo

RUN useradd --uid 1001 --create-home --shell /bin/bash -G sudo,root pimder
RUN echo '%sudo ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers
USER pimder

WORKDIR /home/pimder
ADD . /home/pimder

RUN sudo apt-get install -y  wget curl gzip

RUN mkdir pimd \
    && cd pimd \
    && curl -O https://ccse.jaea.go.jp/software/PIMD/download/pimd.2.7.0.tar.gz \
    && ls -lrth \
    && gzip -dc pimd.2.7.0.tar.gz  | tar -xvzf - 


RUN sudo apt-get install -y  gnupg ca-certificates cmake 

RUN sudo apt install -y  --no-install-recommends tzdata \
    && sudo apt install build-essential -y \
    && sudo apt install emacs git gcc gfortran make patch zip -y \
    && sudo apt install  libfftw3-bin libfftw3-dev libfftw3-doc -y \
    && sudo ln -s /usr/lib/x86_64-linux-gnu/libstdc++.so.6 /usr/lib/x86_64-linux-gnu/libstdc++.so 


RUN wget -O- https://apt.repos.intel.com/intel-gpg-keys/GPG-PUB-KEY-INTEL-SW-PRODUCTS.PUB | gpg --dearmor | sudo tee /usr/share/keyrings/oneapi-archive-keyring.gpg > /dev/null \
    && echo "deb [signed-by=/usr/share/keyrings/oneapi-archive-keyring.gpg] https://apt.repos.intel.com/oneapi all main" | sudo tee /etc/apt/sources.list.d/oneAPI.list \
    && sudo apt-get update -y \
    && sudo apt-get install -y intel-basekit \
    && sudo apt-get install -y intel-hpckit \
    && sudo apt-get install -y intel-oneapi-compiler-fortran \
    && sudo apt-get install -y intel-oneapi-ipp-devel \
    && sudo apt-get install -y intel-oneapi-mpi intel-oneapi-mpi-devel \
    #&& sudo apt-get install -y intel-oneapi-compiler-fortran intel-oneapi-compiler-dpcpp-cpp-and-cpp-classic intel-oneapi-mkl intel-oneapi-mpi intel-oneapi-mpi-devel \
    && echo "source /opt/intel/oneapi/setvars.sh" | cat >> ~/.bashrc 


RUN cd pimd \
    && cp /home/pimder/src.patch lib/aenet/ \
    && . /opt/intel/oneapi/setvars.sh \ 
    && . /opt/intel/oneapi/mkl/latest/env/vars.sh \
    && sudo ln -s $(which ifx) /usr/local/bin/ifort \
    && mkdir build \
    && cd build \
    && export FFT_LIBS="-lfftw3" \
    && export MPIF90=mpiifx \
    && export F90=mpiifx \ 
    && export F77=mpiifx \
    && export FC=mpiifx \
    && export CC=mpiicx \
    && export CXX=mpiicpx \
    && export CFLAGS="-include math.h" \
    && export BLAS_LIBS="-lmkl_intel_lp64  -lmkl_sequential -lmkl_core" \
    && export LAPACK_LIBS="-lmkl_intel_lp64  -lmkl_sequential -lmkl_core" \
    && CC=mpiicx CXX=mpiicpx FC=mpiifx F90=mpiifx F77=mpiifx MPIF90=mpiifx cmake -DMKL_INTERFACE_FULL=intel_lp64  -DQE=on -DQEVERSION=6.3 -DAENET=on -DMKLUSE=on .. \
    && make VERBOSE=1

RUN cd ~/pimd/lib/qe/ \
    && mkdir pseudo \
    && cd pseudo \
    && wget --no-check-certificate http://pseudopotentials.quantum-espresso.org/upf_files/Si.pz-vbc.UPF \
    && wget --no-check-certificate https://pseudopotentials.quantum-espresso.org/upf_files/O.pz-kjpaw.UPF



ENV PATH=/home/pimder/pimd/build/:$PATH


CMD ["bash"]