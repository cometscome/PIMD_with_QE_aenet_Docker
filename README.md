# What is this?
This is Dockerfile for installing PIMD with Quantum Espresso and aenet with Intel OneAPI.

PIMD is an open-source software for parallel molecular simulations: 
https://ccse.jaea.go.jp/software/PIMD/index.en.html

Quantum Espresso is an integrated suite of Open-Source computer codes for electronic-structure calculations and materials modeling at the nanoscale: 
https://www.quantum-espresso.org

The Atomic Energy Network (ænet) is a software package for the construction and usage of atomic interaction potentials based on artificial neural networks (ANNs): 
http://ann.atomistic.net

To build, just do

```bash
docker build --shm-size=2gb -t pimd_oneapi .   
```

After building it, just do

```bash
docker run --shm-size=2gb --name pimd -it pimd_oneapi /bin/bash
```
