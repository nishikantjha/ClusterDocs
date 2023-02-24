# Apptainer on cluster hosts

Apptainer is an open-source container management system that allows users to create, run, and manage containers. Features of the cluster hosts such as centralized identity management, NFS shares and automounting may cause issues with Docker containers. This guide describes discuss some of the basic Singularity commands, as well as some common problems and solutions related to Docker.

## Basic Commands

Apptainer provides three basic commands for container management:

1. `apptainer shell` - This command opens an interactive shell within the container.
2. `apptainer exec` - This command executes commands within the container.
3. `apptainer run` - This command runs a pre-defined runscript within the container.

## Docker problems and solutions by singularity

Singularity provides solutions to some common problems that users may face while working with Docker. Let's discuss these problems and their solutions:

### 1. Executing containers as a regular user:

##### Docker executes containers as root by default, which can be a security issue.

```sh
$ docker run --rm alpine id
uid=0(root) gid=0(root) groups=0(root)
```

##### Singularity runs containers as the same user inside and not root by default. You can use the following command to run containers as a regular user:

```sh
$ singularity shell ubuntu.sif 
Singularity> id
uid=477800155(njha) gid=477800155(njha) groups=477800155(njha),477800051(tio),477800066(docker),477800201(staging)
```
###### You can still run the container as a root user by assigning `--fakeroot` or `-f` while any run, exec, or shell commands.

```sh
$ singularity shell -f ubuntu.sif
Singularity> id
uid=0(root) gid=0(root) groups=0(root)
```

### 2. Mount NFS share directories while running a container

##### Permission denied ERROR using docker:
```sh
$ docker run --rm -it -e POSTGRES_PASSWORD=test -v $HOME/postgresql/data/:/var/lib/postgresql/data postgres
docker: Error response from daemon: error while creating mount source path '$HOMEUSER/postgresql/data': mkdir $HOME/postgresql: permission denied.
```
##### Resolution using singularity:
```sh
singularity instance start -B $HOME/postgresql/data:/var/lib/postgresql/data  postgres_container.sif   postgres  -p 5432
```

## Entering singularity container with particular GPUs using Swarm

```sh
srun --partition GPUMAX --gpus 3,4 --pty -n1 singularity shell --nv pytorch.sif bash
```
