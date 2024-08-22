FROM --platform=linux/x86_64 ubuntu:22.04

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
    && curl -O https://ccse.jaea.go.jp/software/PIMD/download/pimd.2.6.1.tar.gz \
    && ls -lrth \
    && gzip -dc pimd.2.6.1.tar.gz  | tar -xvzf - 

RUN sudo apt-get install -y  gnupg ca-certificates cmake 

RUN wget -O- https://apt.repos.intel.com/intel-gpg-keys/GPG-PUB-KEY-INTEL-SW-PRODUCTS.PUB | gpg --dearmor | sudo tee /usr/share/keyrings/oneapi-archive-keyring.gpg > /dev/null \
    && echo "deb [signed-by=/usr/share/keyrings/oneapi-archive-keyring.gpg] https://apt.repos.intel.com/oneapi all main" | sudo tee /etc/apt/sources.list.d/oneAPI.list \
    && sudo apt-get update -y \
    && sudo apt-get install -y intel-oneapi-compiler-dpcpp-cpp \
    && sudo apt-get install -y intel-oneapi-ipp-devel \
    && sudo apt-get install -y intel-oneapi-compiler-fortran \
    && sudo apt-get install -y intel-oneapi-mkl intel-oneapi-mpi intel-oneapi-mpi-devel \
    #&& sudo apt-get install -y intel-oneapi-compiler-fortran intel-oneapi-compiler-dpcpp-cpp-and-cpp-classic intel-oneapi-mkl intel-oneapi-mpi intel-oneapi-mpi-devel \
    && echo "source /opt/intel/oneapi/setvars.sh" | cat >> ~/.bashrc 
#&& sudo apt-get install -y intel-hpckit 


RUN sudo apt install -y  --no-install-recommends tzdata \
    && sudo apt install emacs git gcc gfortran make patch zip -y \
    && sudo apt install  libfftw3-3 libfftw3-dev libfftw3-doc -y \
    && sudo ln -s /usr/lib/x86_64-linux-gnu/libstdc++.so.6 /usr/lib/x86_64-linux-gnu/libstdc++.so

RUN cd pimd \
    && . /opt/intel/oneapi/setvars.sh \ 
    && mkdir build \
    && cd build \
    && cmake -DQE=on -DQEVERSION=6.3 -DAENET=on -DMKL=on .. \
    && make 

RUN cd ~/pimd/lib/qe/ \
    && mkdir pseudo \
    && cd pseudo \
    && wget --no-check-certificate http://pseudopotentials.quantum-espresso.org/upf_files/Si.pz-vbc.UPF \
    && wget --no-check-certificate https://pseudopotentials.quantum-espresso.org/upf_files/O.pz-kjpaw.UPF



ENV PATH=/home/pimder/pimd/build/:$PATH


CMD ["bash"]