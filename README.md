# NCCL Tests

These tests check both the performance and the correctness of [NCCL](http://github.com/nvidia/nccl) operations.
''This fork of repo is specially for running the nccl-tests without MPI Support''

## Build

To build the tests, just type `make MPI=0` or `make -j MPI=0`

If CUDA is not installed in `/usr/local/cuda`, you may specify `CUDA_HOME`. Similarly, if NCCL is not installed in `/usr`, you may specify `NCCL_HOME`.

```shell
$ make CUDA_HOME=/path/to/cuda NCCL_HOME=/path/to/nccl
```

```shell
$ make MPI=1 NAME_SUFFIX=_mpi MPI_HOME=/path/to/mpi CUDA_HOME=/path/to/cuda NCCL_HOME=/path/to/nccl
```

This will generate test binaries with names such as `all_reduce_perf_mpi`.

## Usage

NCCL tests can run on multiple processes, multiple threads, and multiple CUDA devices per thread. The number of process is managed by MPI and is therefore not passed to the tests as argument. The total number of ranks (=CUDA devices) will be equal to `(number of processes)*(number of threads)*(number of GPUs per thread)`.

### Quick examples

Run on single node with 8 GPUs (`-g 8`), scanning from 8 Bytes to 128MBytes :

```shell
$ ./build/all_reduce_perf -b 8 -e 128M -f 2 -g 8

e.g. 
DOCKER 1

LD_PRELOAD=/usr/local/lib/libnccl.so  NCCL_NET=IB NCCL_P2P_DISABLE=1 NCCL_IB_HCA=roce0 NCCL_IB_GID_INDEX=3 NCCL_SOCKET_IFNAME=eth0 NCCL_IB_DISABLE=0 NCCL_DEBUG=INFO NCCL_COMM_ID=20.20.20.2:12345 WORLD_SIZE=2 RANK=0 /nvidia-tools/nccl-tests-no-ompi/build/all_reduce_perf -b 4M -e 4M

DOCKER 2
LD_PRELOAD=/usr/local/lib/libnccl.so  NCCL_NET=IB NCCL_P2P_DISABLE=1 NCCL_IB_HCA=roce0 NCCL_IB_GID_INDEX=3 NCCL_SOCKET_IFNAME=eth0 NCCL_IB_DISABLE=0 NCCL_DEBUG=INFO NCCL_COMM_ID=20.20.20.2:12345 WORLD_SIZE=2 RANK=1 /nvidia-tools/nccl-tests-no-ompi/build/all_reduce_perf -b 4M -e 4M

```

Note:
If you are planning to run this on docker on same hosts you will get this message

**31a6b39aaaf4:367:372 [0] misc/socket.cc:432 NCCL WARN socketFinalizeAccept: wrong magic ba8657f4d3a5a02a != 134eccad5a34ad33**

To Resolve build nccl with this patch below. 
and then load this nccl before running nccl-tests ( LD_PRELOAD=/usr/local/lib/libnccl.so)

```
+++ b/src/misc/socket.cc
@@ -481,11 +481,12 @@ static ncclResult_t socketFinalizeAccept(struct ncclSocket* sock) {
       memcpy(&magic, sock->finalizeBuffer, sizeof(magic));
     }
     if (magic != sock->magic) {
-      WARN("socketFinalizeAccept: wrong magic %lx != %lx", magic, sock->magic);
-      close(sock->fd);
-      sock->fd = -1;
+      WARN("socketFinalizeAccept2: wrong magic %lx != %lx", magic, sock->magic);
+      //close(sock->fd);
+      //sock->fd = -1;
       // Ignore spurious connection and accept again
-      sock->state = ncclSocketStateAccepting;
+      //sock->state = ncclSocketStateAccepting;
+      sock->state = ncclSocketStateReady;
       return ncclSuccess;
```


