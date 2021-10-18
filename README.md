# What is this?
This is Dockerfile for installing PIMD with Quantum Espresso and aenet with Intel OneAPI.

To build, just do

```bash
docker build --shm-size=2gb -t pimd_oneapi .   
```

After building it, just do

```bash
docker run --shm-size=2gb --name pimd -it pimd_oneapi /bin/bash
```
