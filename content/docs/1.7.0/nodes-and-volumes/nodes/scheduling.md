---
title: Scheduling
weight: 5
---

In this section, you'll learn how Longhorn schedules replicas based on multiple factors.

### Scheduling Policy

Longhorn's scheduling policy has two stages. The scheduler only goes to the next stage if the previous stage is satisfied. Otherwise, the scheduling will fail.

If any tag has been set in order to be selected for scheduling, the node tag and the disk tag have to match when the node or the disk is selected.

The first stage is the **node and zone selection stage.** Longhorn will filter the node and zone based on the `Replica Node Level Soft Anti-Affinity` and `Replica Zone Level Soft Anti-Affinity` settings.

The second stage is the **disk selection stage.** Longhorn will filter the disks that satisfy the first stage based on the `Replica Disk Level Soft Anti-Affinity`, `Storage Minimal Available Percentage`, `Storage Over Provisioning Percentage`, and other disk-related factors like requested disk space.

#### The Node and Zone Selection Stage

First, Longhorn will always try to schedule the new replica on a new node with a new zone if possible. In this context, "new" means that a replica for the volume has not already been scheduled to the zone or node, and "existing" refers to a node or zone that already has a replica scheduled to it.

At this time, if both the `Replica Node Level Soft Anti-Affinity` and `Replica Zone Level Soft Anti-Affinity` settings are un-checked, and if there is no new node with a new zone, Longhorn will not schedule the replica.

Then, Longhorn will look for a new node with an existing zone. If possible, it will schedule the new replica on the new node with an existing zone.

At this time, if `Replica Node Level Soft Anti-Affinity` is un-checked and `Replica Zone Level Soft Anti-Affinity` is checked, and there is no new node with an existing zone, Longhorn will not schedule the replica.

Last, Longhorn will look for an existing node with an existing zone to schedule the new replica. At this time both `Replica Node Level Soft Anti-Affinity` and `Replica Zone Level Soft Anti-Affinity` should be checked.

#### Disk Selection Stage

Once the node and zone stage is satisfied, Longhorn will decide whether it can schedule the replica on any disk of the node. Longhorn will check the available disks on the selected node with the matching tag, the total disk space, and the available disk space. It will also check whether another replica already exists and whether anti-affinity is set to be "hard" (no sharing) or "soft" (prefer not to share.)

For example, after the node and zone stage, Longhorn finds `Node A` satisfies the requirements for scheduling a replica to the node. Longhorn will check all the available disks on this node.

Assume this node has two disks and neither one has another replica: `Disk X` with available space 1 GB, and `Disk Y` with available space 2 GB. And the replica Longhorn going to schedule needs 1 GB. With default `Storage Minimal Available Percentage` 25, Longhorn can only schedule the replica on `Disk Y` if this `Disk Y` matches the disk tag, otherwise Longhorn will return failure on this replica selection. But if the `Storage Minimal Available Percentage` is set to 0, and `Disk X` also matches the disk tag, Longhorn can schedule the replica on `Disk X`.

Now suppose one of the potential candidate disks has an existing replica and `Replica Disk Soft Anti-Affinity" is set to true.  In principle, Longhorn would be allowed to choose either disk, but in practice, it will avoid the existing replica and place the new replica on another disk, even if it is an otherwise inferior choice.

### Settings

For more information on settings that are relevant to scheduling replicas on nodes and disks, refer to the settings reference:

- [Disable Scheduling On Cordoned Node](../../../references/settings/#disable-scheduling-on-cordoned-node)
- [Replica Soft Anti-Affinity](../../../references/settings/#replica-node-level-soft-anti-affinity) (also called Replica Node Level Soft Anti-Affinity)
- [Replica Zone Level Soft Anti-Affinity](../../../references/settings/#replica-zone-level-soft-anti-affinity)
- [Replica Disk Level Soft Anti-Affinity](../../../references/settings/#replica-disk-level-soft-anti-affinity)
- [Storage Minimal Available Percentage](../../../references/settings/#storage-minimal-available-percentage)
- [Storage Over Provisioning Percentage](../../../references/settings/#storage-over-provisioning-percentage)
- [Allow Empty Node Selector Volume](../../../references/settings/#allow-empty-node-selector-volume)
- [Allow Empty Disk Selector Volume](../../../references/settings/#allow-empty-disk-selector-volume)

### Notice
Longhorn relies on label `topology.kubernetes.io/zone=<Zone name of the node>` or `topology.kubernetes.io/region=<Region name of the node>` in the Kubernetes node object to identify the zone/region.

Since these are reserved and used by Kubernetes as [well-known labels](https://kubernetes.io/docs/reference/labels-annotations-taints/#topologykubernetesiozone).
