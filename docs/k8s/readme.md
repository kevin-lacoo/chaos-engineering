# **Module: `k8s`**

## Purpose

### What?

This module provides [`chaostoolkit`](https://chaostoolkit.org) actions to disrupt pods in Kubernetes clusters. It supports:

- **Pods**: *Termination* of pods in one zone with a min/max lifetime (e.g. 0s-0s to shoot down any pod right when it tries to come up or e.g. 10-60s to let them come up at least for 10s but shoot them down at the latest after 60s) with or without a grace period.

You can run the above in parallel, as long as the targeted zones differ. This way you can also test whether you recover after a multi-zonal outage.

This module also provides [`chaostoolkit`](https://chaostoolkit.org) probes:

- **Health Probe**: Probes various Kubernetes cluster functions in parallel.

:warning: The probe requires resources that have not yet been shared as of today (dynamic cert generation must be implemented first), so it isn't usable today for you, unless you have also access to said resources.

### How?

- **Pods**: Based on the given zone and filters, pods are identified busily/continuously and *terminated* with or without a grace period. You may provide a min/max lifetime to make the process more random, chaotic, and unpredictable, which may further help you unearth issues.
- **Health Probe**: Deploys probes into the cluster that busily/continuously probe various Kubernetes cluster functions in parallel. This operation must be rolled back when completed.

### Why?

Implementing high availability that can even tolerate a zone outage unscathed is no trivial task. You can find more information on how to achieve this goal [here](/docs/k8s/high-availability.md). To put your solution to the test, this module will help you.

The probe on the other hand is targeting Kubernetes provider developers and output-qualification and puts cluster HA as such to the test, which requires automation as Kubernetes clusters may perform many functions in parallel.

## Usage

### Actions and Rollbacks

`chaostoolkit` introduces so-called [actions](https://chaostoolkit.org/reference/api/experiment/#action) that can be composed into [experiments](https://chaostoolkit.org/reference/api/experiment/#experiment) that perform operations against a system (here a Kubernetes cluster). The following actions (and explicit [rollbacks](https://chaostoolkit.org/reference/api/experiment/#rollbacks)) are supported:

Module: [`chaosgarden.k8s.actions`](/chaosgarden/k8s/actions.py)

- `run_pod_failure_simulation`: Run pod failure simulation.
- `run_pod_failure_simulation_in_background`: Same as above, but running in background as a thread. Normally not used with experiments, but directly in Python (scripts).

- `run_cluster_health_probe`: Run cluster health probe (usually only interesting to Kubernetes provider developers).
- `rollback_cluster_health_probe`: Rollback cluster health probe explicitly (usually performed automatically above, but can also be invoked explicitly as rollback step in an experiment to deal with interruptions).
- `run_cluster_health_probe_in_background`: Same as above, but running in background as a thread. Normally not used with experiments, but directly in Python (scripts).

### Pod Selectors

The following pod selectors are supported:

- `node_label_selector`, e.g. `topology.kubernetes.io/zone=world-1a,worker.gardener.cloud/pool=cpu-worker,...`, right-hand side may be a regex, operators are `=|==|!=|=~|!~`
- `pod_label_selector`, e.g. `gardener.cloud/role=controlplane,gardener.cloud/role=vpa,...`, regular [pod label selector](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/#label-selectors) (not interpreted by `chaosgarden`)
- `pod_metadata_selector`, e.g. `namespace=kube-system,name=kube-apiserver.*,...`, right-hand side may be a regex, operators are `=|==|!=|=~|!~`
- `pod_owner_selector`, e.g. `kind!=DaemonSet,name=kube-apiserver.*,...`, right-hand side may be a regex, operators are `=|==|!=|=~|!~`

### Configuration

No [configuration](https://chaostoolkit.org/reference/api/experiment/#configuration) required.

### Secrets

The following [secret](https://chaostoolkit.org/reference/api/experiment/#secrets) fields are optional (only one is permitted; if none is set, `$KUBECONFIG` is assumed to be pointing to the cluster):

- `kubeconfig_struct`: Kubernetes cluster configuration for cluster (json struct)
- `kubeconfig_file`: Kubernetes cluster configuration for cluster (path to kubeconfig file)
- `kubeconfig_envvar`: Kubernetes cluster configuration for cluster (env var with path to kubeconfig file)

## Examples

- [Run Pod Failure Simulation](/docs/k8s/run-pod-failure-simulation.json)

- [Run Cluster Health Probe as Hypothesis](/docs/k8s/run-cluster-health-probe-as-hypothesis.json) (doesn't really fit as it must run in background, which is not supported by `chaostoolkit`)
- [Run Cluster Health Probe as Method](/docs/k8s/run-cluster-health-probe-as-method.json) (the better alternative and almost identical in `chaostoolkit` behavior)

- [Explicit Kubernetes Secrets](/docs/k8s/explicit-k8s-secrets.json) (if you do not want to use `$KUBECONFIG` pointing to the cluster)