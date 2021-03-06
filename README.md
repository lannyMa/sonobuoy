# Sonobuoy

**Maintainers:** [Heptio][0]

[![Build Status][1]][2]


## Overview

Heptio Sonobuoy is a diagnostic tool that makes it easier to understand the state of a Kubernetes cluster by running a set of [Kubernetes][3] conformance tests in an accessible and non-destructive manner. It is a customizable, extendable, and cluster-agnostic way to generate clear, informative reports about your cluster--regardless of your deployment details.

Its selective data dumps of Kubernetes resource objects and cluster nodes allow for the following use cases:

* Integrated end-to-end (e2e) [conformance-testing 1.7+][13]
* Workload debugging
* Custom data collection via extensible plugins

## Supported Versions

*NOTE: The table below only applies to the e2e conformance tests.*

To ensure that your cluster is running the appropriate Kubernetes version for your Sonobuoy release, see the second column of the table below. Otherwise you may encounter the documented issues.

| Sonobuoy Version | Supported K8s Cluster Versions | Note(s) |
|---|---|---|
| master | 1.9 <= X <= TBD | Under development |
| v0.10.x | 1.7 <= X <= 1.9 | Verify your kube-conformance container version matches cluster version |

## Prerequisites

* **You should have access to an up-and-running Kubernetes cluster.** If you do not have a cluster, follow the [AWS Quickstart Kubernetes Tutorial][5] to set one up with a single command.

* **You should have `kubectl` installed.** If not, follow the instructions for [installing via Homebrew (MacOS)][6] or [building the binary (Linux)][7].

* **You must run the `kubectl apply` commands for Sonobuoy as an admin user.**

* **Your firewall settings must allow Sonobuoy to communicate with your cluster.**

* **Your cluster must have the appropriate RBAC ClusterRole and ClusterRoleBinding configured.** By default, the Sonobuoy YAML assumes that your cluster is running with `--authorization-mode=RBAC`, and includes the required RBAC configurations.

  (If you don't have RBAC enabled, subsequent documentation addresses how to handle this.)

## Up and running

To easily get a Sonobuoy scan started on your cluster, use the browser-based [Sonobuoy Scanner tool][18]. Sonobuoy Scanner also provides a more user-friendly way of viewing your scan results.

*Note that Sonobuoy Scanner runs conformance tests only. To learn how to run Sonobuoy with your own YAML files, which allows for further configuration and customization, see the [Quickstart][19] section below.*

![tarball overview screenshot][20]


## Quickstart

> Heptio provides prebuilt Sonobuoy container images in its Google Container Registry (*gcr.io/heptio-images*). For the sake of faster setup on your cluster, **this quickstart pulls from this registry to skip the container build process**. You can use this same process to deploy Sonobuoy to production.


This guide executes a Sonobuoy run on your cluster, and records the following results:
* Basic info about your cluster's hosts, Kubernetes resources, and versions.
* *(Via plugin)* [`systemd`][14] logs from each host
* *(Via plugin)* The results of a single e2e conformance test ("Pods should be submitted and removed"). See the [conformance guide][13] for configuration details.

### 1. Download
Clone or fork the Sonobuoy repo:
```
git clone https://github.com/heptio/sonobuoy.git
```

### 2. Run

First, make sure that you're in your Sonobuoy root directory.

If your cluster is not running RBAC, you'll need to regenerate the quickstart YAML files. To determine whether your cluster is running RBAC, run the command below:

```
export RBAC_ENABLED=$(kubectl api-versions | grep "rbac.authorization.k8s.io/v1" -c)
```

Use the environment variable you just set to regenerate `examples/quickstart.yaml`:

```
make generate
```

Now you're ready to deploy a Sonobuoy pod to your cluster! Run the following command:
```
kubectl apply -f examples/quickstart.yaml
```

You can view actively running pods with the following command:
```
kubectl get pods -l component=sonobuoy --namespace=heptio-sonobuoy
```

To verify that Sonobuoy has completed successfully, check the logs:
```
kubectl logs -f sonobuoy --namespace=heptio-sonobuoy
```
If you see the log line `no-exit was specified, sonobuoy is now blocking`, the Sonobuoy pod has completed its data collection.

> *Notes*:
>
> * **The Sonobuoy pod in this example continues to run after it finishes data collection**, due to the `--no-exit` flag in its YAML manifest. This allows you to easily grab the results tarball.
>
> * **Sonobuoy collects one data report per run**---each time you want a new report you need to delete the existing Sonobuoy YAML files and reapply them (see the [tear down step][15]).
>
> * **In practice, you should make sure that the Sonobuoy pod writes its results to a Persistent Volume.** The quickstart example writes its output to an `emptyDir` volume for simplicity.
>
> *Troubleshooting errors from `kubectl logs`*:
>  * If you are able to debug and resolve the issue on your own, *make sure to delete and reapply Sonobuoy's YAML manifests*. Otherwise, [file an issue][10].
>

To view the output, copy the output directory from the main Sonobuoy pod to somewhere local:
```
kubectl cp heptio-sonobuoy/sonobuoy:/tmp/sonobuoy ./archive --namespace=heptio-sonobuoy
```

This should copy a single `.tar.gz` snapshot from the Sonobuoy pod into your local `./results` directory. You can extract its contents into `/.results` with:
```
tar xzf ./results/*.tar.gz
```

For information on the contents of the snapshot, see the [snapshot documentation](docs/snapshot.md).

*NOTE: At this time, the layout of the contents of the tarball is subject to change.*

### 3. Tear down

To clean up Kubernetes objects created by Sonobuoy, run the following command:
```
kubectl delete -f examples/quickstart.yaml
```

## Further documentation

 To learn how to configure your Sonobuoy runs and integrate plugins, see the [`/docs` directory][9].

## Troubleshooting

If you encounter any problems that the documentation does not address, [file an issue][10].

## Contributing

Thanks for taking the time to join our community and start contributing!

#### Before you start

* Please familiarize yourself with the [Code of
Conduct][12] before contributing.
* See [CONTRIBUTING.md][11] for instructions on the
developer certificate of origin that we require.
* There is a [mailing list][16] and [Slack channel][17] if you want to interact with
other members of the community

#### Testing

You can run sonobuoy's tests with `make test`. This will spin up local docker
containers to run the test suite.

For noiser tests, use `VERBOSE=true make test`

#### Pull requests

* We welcome pull requests. Feel free to dig through the [issues][10] and jump in.

## Changelog

See [the list of releases](https://github.com/heptio/sonobuoy/releases) to find out about feature changes.

[0]: https://github.com/heptio
[1]: https://jenkins.i.heptio.com/buildStatus/icon?job=sonobuoy-deployer
[2]: https://jenkins.i.heptio.com/job/sonobuoy-deployer/
[3]: https://github.com/kubernetes/kubernetes
[5]: http://docs.heptio.com/content/tutorials/aws-cloudformation-k8s.html
[6]: https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-with-homebrew-on-macos
[7]: https://kubernetes.io/docs/tasks/tools/install-kubectl/#tabset-1
[8]: https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/
[9]: /docs
[10]: https://github.com/heptio/sonobuoy/issues
[11]: /CONTRIBUTING.md
[12]: /CODE_OF_CONDUCT.md
[13]: /docs/conformance-testing.md
[14]: https://github.com/systemd/systemd
[15]: #3-tear-down
[16]: https://groups.google.com/forum/#!forum/heptio-sonobuoy
[17]: https://kubernetes.slack.com/messages/sonobuoy
[18]: https://scanner.heptio.com/
[19]: #quickstart
[20]: docs/img/scanner.png
