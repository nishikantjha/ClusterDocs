# Singularity on cluster hosts

Features of the cluster hosts such as centralized identity management, NFS shares and automounting may cause issues with Docker containers. This guide describes how to work around the most common problems using SIngularity.

# Docker problems and solutions by singularity
`
## Executing containers as a regular user

```sh
$ id
uid=477800155(njha) gid=477800155(njha) groups=477800155(njha),477800051(tio),477800066(docker),477800201(staging)
```
##### Containers are executed as root by default

```sh
$ docker run --rm alpine id
uid=0(root) gid=0(root) groups=0(root),1(bin),2(daemon),3(sys),4(adm),6(disk),10(wheel),11(floppy),20(dialout),26(tape),27(video)
```
##### The root user problem is solved by launching the container as the current user and primary group:

```sh
$ docker run --rm --user="$(id -u):$(id -g)" alpine id
uid=477800155 gid=477800155 groups=477800155
```


##### Singularity containers are executed as the same user inside and not root by default
```sh
$ singularity shell ubuntu.sif 
Singularity> id
uid=477800155(njha) gid=477800155(njha) groups=477800155(njha),477800051(tio),477800066(docker),477800201(staging)
```
##### Also, you can still run as root user by assigning --fakeroot or -f while any run, exec or shell commands

```sh
$ singularity shell -f ubuntu.sif
Singularity> id
uid=0(root) gid=0(root) groups=0(root)
```

## Mount NFS share directories while running a container

##### Permission denied ERROR using docker:
```sh
$ docker run --rm -it -e POSTGRES_PASSWORD=test -v /homes/njha/postgresql/data/:/var/lib/postgresql/data postgres
docker: Error response from daemon: error while creating mount source path '/homes/njha/postgresql/data': mkdir /homes/njha/postgresql: permission denied.
```
##### Resolution using singularity:
```sh
SINGULARITYENV_POSTGRES_PASSWORD=local_password singularity instance start -B $HOME/postgresql/data:/var/lib/postgresql/data   postgres_container.sif   postgres  -p 5432
```

##### Entering singularity container with particular GPUs using docker swarm

```sh
srun --partition GPUMAX --gpus 3,4 --pty -n1 singularity shell --nv pytorch.sif bash
```
