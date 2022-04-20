# Migration from v0.0.15 to 1.0

From XXX, we have made one breaking change:
- renaming the plugin from `ebs.csi.aws.com` to `csi.outscale.com`

To make things smoothe, here a guide to migrate from the two versions

## Installation
The difference between the installation explained in the [Deploy](./deploy.md) for a standard upgrade and for this migration is that we need to change the `liveness` port of the `csi-node` pod. To do that, add the following in the helm command
```shell
--set sidecars.livenessProbeImage.port=9809
```

This is to ensure that it would not have conflict between pods from both plugins.

## Migrate the Volume
The next step is to migrate all volume from using the old csi driver to the new one. The approach that we will explain is to make snapshot from the previous volumeand create a new one from the snapshot.

1. Scale all the application to 0
> **_Warning:_** Before shutting down the pod that uses volumes, you need to make sure that the `Retain Policy` is set to `Retain`
2. Create the `Snapshot Class`
   > See this [example](../examples/kubernetes/snapshot/specs/classes/storageclass.yaml)
3. Make a snapshot from the PVC
   > Change the `persistentVolumeClaimName` from this example [example](../examples/kubernetes/snapshot/specs/snapshot/snapshot.yaml)
4. Create the PVC from this snapshot
   > Change the `dataSource.name` from this [example](../examples/kubernetes/snapshot/specs/snapshot-restore/claim.yaml) 
5. Change the PVC fror the pod and scale up again

Once all the volume have been migrated and you check that it work, you can uninstall the old driver.
