# Migration from OpenShift to Anthos/GKE Cluster
[DRAFT]

This guide provides scripts and tools to migration cluster configurations and workloads from an OpenShift Cluster to Anthos or GKE Cluster.

## Migrating OpenShift Project Configurations

This section addresses migrating openshift projects, cluster level configurations and project level configurations to the target cluster. This process is semi-automated because certain decisions require choices to be made by the person who is migrating. Also the procedure allows you to either migration an application at a time or all the workloads running on a cluster.

### Prerequisites

* Linux bash shell: These scripts have been tested on google cloud console
* `oc` - [openshift client](https://docs.openshift.com/container-platform/4.7/cli_reference/openshift_cli/getting-started-cli.html#installing-openshift-cli). Login to the OpenShift cluster from which you are migrating.
* `yq` - [yaml processing tool](https://github.com/mikefarah/yq#install)

### Export Project Configurations

* [Export Projects to Namespaces](1.ExportingProjects.md)
* [Export ClusterResourceQuotas to K8S ResourceQuotas](2.ClusterResourceQuota.md)
* [Export ClusterRoles and ClusterRoleBindings](3.ClusterRolesAndRoleBindings.md)
* [Capture NetNamespaces data](4.NetNameSpaces.md)
* [Export Project Level Resource Quotas](5.ResourceQuotas.md)
* [Export Project Level Service Accounts, Roles and RoleBindings](6.RolesAndRoleBindings.md)
* [Capture Egress Network Policies data](7.EgressNetworkPolicies.md)
* Apply NetworkPolicies based on EgressNetworkPolicies (WIP)

**All the steps in the above documentation links should be read.** Once you read and understand, you can run the following scripts rather than individually copy pasting the scripts. These scripts will generate a folder named `clusterconfigs` with the manifests that can be applied to the target GKE Cluster. The folder structure follows [ACM repo layout](https://cloud.google.com/kubernetes-engine/docs/add-on/config-sync/concepts/repo). You can create a git repo and apply this using [ConfigSync](https://cloud.google.com/kubernetes-engine/docs/add-on/config-sync/overview) to an Anthos Cluster.

* Run script#1 that exports namespaces, clusterresourcequotas and cluster roles.

```
chmod +x ./scripts/migrateScript1.sh
./scripts/migrateScript1.sh
```
* Review the namespace manifests generated in the `clusterconfigs/namespaces` and remove the ones that don't need to be migrated
* Review ClusterRoles in the `clusterconfigs/cluster/cluster-roles` folder and remove the ones that don't need to be migrated
* Run script#2 to export ClusterRoleBindings and namespace Level configurations.

```
chmod +x ./scripts/migrateScript2.sh
./scripts/migrateScript2.sh
```
* Review the Service Accounts, Roles and RoleBindings that are generated in the individual namespace folders and remove the ones that don't need to be migrated to the target cluster
* Review ClusterResourceQuotas in `clusterconfigs/to-review/cluster-resource-quotas` and copy the templates to create namespace specific quotas in the namespace folders with namespace based allocations
* Review NetNamespaces in the `clusterconfigs/to-review/net-namespaces` folder. **Handling TBD**

## Deploy Configurations with Anthos Config Manager (ACM)

* Stand up a Anthos cluster if you don't already have one
* [Install nomos](https://cloud.google.com/kubernetes-engine/docs/add-on/config-sync/how-to/nomos-command)
* Initialize `clusterconfigs` repo  
```
cd clusterconfigs
nomos init --force
```
* Create a git repository to host the structure. Initialize the `clusterconfig` folder as a git repo and push the to git repo.
* [Install ConfigSync](https://cloud.google.com/kubernetes-engine/docs/add-on/config-sync/how-to/installing). Configure the git repo as the Sync Repo
* Login to the Anthos Cluster and verify that the manifests from the repository are applied.

## Migrating OpenShift SCCs to ACM Constraints
WIP

## Migrating Workloads to Target GKE Cluster
WIP

## Migrate Persistent Data
WIP
