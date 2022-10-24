FROM --platform=linux/x86_64 ubuntu:20.04

RUN apt-get update
RUN apt-get install -y sudo

RUN useradd --uid 1001 --create-home --shell /bin/bash -G sudo,root pimder
RUN echo '%sudo ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers
USER pimder

WORKDIR /home/pimder
ADD . /home/pimder

RUN sudo apt update -y; sudo apt install -y wget gnupg \
 && cd /tmp \
&& sudo wget https://apt.repos.intel.com/intel-gpg-keys/GPG-PUB-KEY-INTEL-SW-PRODUCTS.PUB \
&& sudo apt-key add GPG-PUB-KEY-INTEL-SW-PRODUCTS.PUB \
&& sudo rm GPG-PUB-KEY-INTEL-SW-PRODUCTS.PUB \
&& echo "deb https://apt.repos.intel.com/oneapi all main" | sudo tee /etc/apt/sources.list.d/oneAPI.list \
&& sudo apt update -y \
&& sudo apt install intel-oneapi-compiler-fortran intel-oneapi-compiler-dpcpp-cpp-and-cpp-classic \
    intel-oneapi-mkl intel-oneapi-mpi intel-oneapi-mpi-devel -y \
&& echo "source /opt/intel/oneapi/setvars.sh" | cat >> ~/.bashrc  

RUN sudo apt install -y --no-install-recommends tzdata \
    && sudo apt install emacs git gcc gfortran make patch zip -y \
    && sudo apt install libfftw3-3 libfftw3-dev libfftw3-doc -y \
    && sudo ln -s /usr/lib/x86_64-linux-gnu/libstdc++.so.6 /usr/lib/x86_64-linux-gnu/libstdc++.so

RUN mkdir pimd \
 && cd pimd \
 && . /opt/intel/oneapi/setvars.sh \
 && wget https://ccse.jaea.go.jp/software/PIMD/download/pimd.2.5.2.tar \
 && tar xvfz pimd.2.5.2.tar \
 && cp -r source compile \
 && cd compile \
 && cp /home/pimder/makefile_qe.patch . \
 && patch -u makefile < makefile_qe.patch \
 && make common_variables.o 

# install QE
RUN cd ~/pimd/lib/qe \
 && wget https://github.com/QEF/q-e/archive/qe-6.2.1.zip \
 && . /opt/intel/oneapi/setvars.sh \
 && unzip qe-6.2.1.zip \
 && patch -p0 < qe-6.2.1_to_pimdlib.patch \
 && cd ~/pimd/lib/qe/q-e-qe-6.2.1 \
 && ./configure MPIF90=mpiifort F90=ifort F77=ifort CC=gcc --enable-parallel --with-scalapack=intel \
 #&& ./configure \
 && cd ~/pimd/lib/qe/q-e-qe-6.2.1 \
 && make pw 

# install aenet
RUN cd ~/pimd/lib/aenet/ \
&& git clone https://github.com/atomisticnet/aenet.git \
&& . /opt/intel/oneapi/setvars.sh \
&& cp -r aenet/src/ . && cp -r aenet/lib/ . \
&& cp /home/pimder/Makefile_aenetlib.patch ./lib/ \
&& cd lib/ \
&& patch -u Makefile < Makefile_aenetlib.patch \
&& make && cp liblbfgsb.a ~/pimd/lib/ \
&& cd ~/pimd/lib/aenet/ \
&& cp -r src src_modified \ 
&& patch -u -p1 -d src_modified < src.patch \
&& cd src_modified/ \
&& make -f makefiles/Makefile.ifort_intelmpi lib \
&& cp libaenet.a ~/pimd/lib/ \
&& cd ../../../

RUN cd ~/pimd/lib/qe/ \
 && mkdir pseudo \
 && cd pseudo \
 && wget http://pseudopotentials.quantum-espresso.org/upf_files/Si.pz-vbc.UPF \
 && wget https://pseudopotentials.quantum-espresso.org/upf_files/O.pz-kjpaw.UPF


# install pimd with QE and aenet
RUN cd ~/pimd/compile/ \
&& . /opt/intel/oneapi/setvars.sh \
&& make pimd.mpi.x

ENV PATH=/home/pimder/pimd/compile/:$PATH


CMD ["bash"]