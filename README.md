### Introduction

This is a simple demonstration of using user namespaces in Pipelines to support test containers.

The maven task runs a rootless podman sidecar, so that the Testcontainers based
tests in the sample application can start a postgresql container from inside the
build pod. The pod runs with `hostUsers: false`, which means those nested
containers run entirely inside the pod's user namespace and are unprivileged on
the host.

Everything is built from resources that ship with OpenShift and OpenShift
Pipelines:

* The OpenShift provided `nested-container` SCC, bound to the pipeline service
  account through the auto generated `system:openshift:scc:nested-container`
  cluster role. No custom SCC is defined.
* Red Hat provided images, `ubi9/openjdk-21` for maven and `rhel9/podman` for
  the sidecar. No image needs to be built for this demo.
* The `git-clone` task from the `openshift-pipelines` namespace.

### Prerequisites

* OpenShift Cluster 4.21+
* OpenShift Pipelines 1.24+

### Deploy Example

Deploy the demo using the following command. The only cluster scoped permission
needed is the ability to grant the `nested-container` SCC:

```
oc apply -k .
```

### Run Pipeline

To run the pipeline, use the provided pipelinerun-user-namespaces.yaml file:

```
oc create -f pipelinerun-user-namespaces.yaml
```

### Notes

* The podman sidecar needs `/dev/net/tun` so that pasta can give each nested
  container its own network namespace with published ports, and `/dev/fuse` so
  that fuse-overlayfs can be used instead of the much slower vfs storage driver.
  Both are requested with the `io.kubernetes.cri-o.Devices` annotation on the
  task, which Tekton copies onto the TaskRun pod.
* Only the podman sidecar is granted the SETUID and SETGID capabilities and an
  unmasked `/proc`. The maven step itself runs with the default restricted
  capability set and just talks to the podman socket over `DOCKER_HOST`.
* The local maven repository is an emptyDir, so each run downloads its
  dependencies. Backing it with a PVC is possible but Tekton's affinity
  assistant rejects a TaskRun that binds more than one PVC, so it would have to
  share a PVC with the source workspace via subPaths, or the cluster would need
  `coschedule: pipelineruns` set in the TektonConfig.
