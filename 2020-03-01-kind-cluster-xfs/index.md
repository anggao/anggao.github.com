# Kind Cluster issue


To use Kind cluster you need to have overlay(2) storage driver with Docker if you are using XFS filesystem.

## overlayfs-driver

https://docs.docker.com/storage/storagedriver/overlayfs-driver/

`The overlay and overlay2 drivers are supported on xfs backing filesystems, but only with d_type=true enabled.`

```
xfs_info /

meta-data=/dev/mapper/centos_agao isize=512    agcount=4, agsize=3276800 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0 spinodes=0
data     =                       bsize=4096   blocks=13107200, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal               bsize=4096   blocks=6400, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
```
The 3rd column of the 6th line of the xfs_info output is the most interesting because it contains the parameter ftype which should be 1. When ftype is 0, d_type support is disabled. When it is 1, d_type support is enabled and you're safe to use the overlay(2) storage driver with Docker on an XFS filesystem.

### K8s event

```
kubectl get event
```

If you go the error

```
Failed to update Node Allocatable Limits ["kubepods"]: failed to set supported cgroup subsystems for cgroup [kubepods]: failed to find subsystem mount for required subsystem: pids
```
Check whehter your kernel supports pid subsystem, if not you need to upgrade your linux kernel.
```
cat /proc/cgroups
```

