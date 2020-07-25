---
title: Kind Cluster issue
draft: false
date: "2019-02-21"
tags:
  - k8s
---
https://docs.docker.com/storage/storagedriver/overlayfs-driver/

`The overlay and overlay2 drivers are supported on xfs backing filesystems, but only with d_type=true enabled.`

xfs_info /

`ascii-ci=0 ftype=0`

The 3rd column of the 6th line of the xfs_info output is the most interesting because it contains the parameter ftype which should be 1. When ftype is 0, d_type support is disabled. When it is 1, d_type support is enabled and you're safe to use the overlay(2) storage driver with Docker on an XFS filesystem.
