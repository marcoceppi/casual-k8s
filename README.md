# Production-like minikube

Minikube is a great project for piecing together a single VM kubernetes infrastructure. It, however, falls
short when trying to seriously evaluate kubernetes or replicate what would normally run in production on a
local system.

The aim of this project is to replicate the ease of minikube with the quality and topology of a robust, and
scalable, Kubernetes deployment. The scope of this project is to produce the right infrastructure on a local
Windows, OSX, or Linux desktop/laptop and passing those VMs off as "machines". Those machines can then be used
as if they were physical bare metal, or an instance in a cloud but higher level tooling. Instead of building a
tool specifically for deploying kubernetes on VMs, we're building the layer that produces VMs that existing
tooling can leverage.

The goal is, ultimately, to provide developers and users with a real world example of Kubernetes running in
toplogies expected for production on the machine the developer is using.

# Design

Starting, this tool will be able to create VM topologies for two types of deployment.

## Kubernetes Core

This is a small, two node kubernetes cluster with the master and worker nodes separated. The vast majority of
Kubernetes deployments, today, perform a separation of master from worker. A core deployment will always have
a minimum of two VMs as this is where we separate casual-k8s from minikube.

* Kubernetes Master
* Kubernetes Worker
* EasyRSA Certificate Authority
* Flannel SDN
* ETCD


```
┌───────────────┐    ┌───────────────┐
│ VM01          │    │ VM02          │
├───────────────┤    ├───────────────┤
│ * EasyRSA     │    │ * K8S Worker  │
│ * ETCD        ├────┤               │
│ * K8S Master  │    │               │
└───────────────┘    └───────────────┘
```

This will be accomplished with any of the following commands:

    casual-k8s deloy kubernetes-core [DEPLOYMENT-NAME]
    casual-k8s deloy k8s-core [DEPLOYMENT-NAME]
    casual-k8s deloy core [DEPLOYMENT-NAME]


## Kubernetes

The second toplogy is a variant of core which spreads the applications in question across concerns and
makes the kubernetes master scalable and HA. This will invoke a minimum of four VMs.

* Kubernetes API Loadbalancer
* Kubernetes Master
* Kubernetes Worker
* EasyRSA Certificate Authority
* Flanne SDN
* ETCD

```
                     ┌───────────────┐
                     │ VM01          │
                     ├───────────────┤
                     │ * EasyRSA     │
                     │ * ETCD        │
                     │ * API LB      │
                     └───────┬───────┘
                             │
        ┌────────────────────┼────────────────────┐
        │                    │                    │
┌───────┴───────┐    ┌───────┴───────┐    ┌───────┴───────┐
│ VM02          │    │ VM03          │    │ VM03          │
├───────────────┤    ├───────────────┤    ├───────────────┤
│ * K8S Master  │    │ * K8S Master  │    │ * K8S Worker  │
│               │    │               │    │               │
│               │    │               │    │               │
└───────────────┘    └───────────────┘    └───────────────┘
```

This will be accomplished with any of the following commands:

    casual-k8s deloy kubernetes [DEPLOYMENT-NAME]
    casual-k8s deloy k8s [DEPLOYMENT-NAME]

# Usage

This tool provides a yet to be named command-line interface for Linux, OSX, and Windows. The place
holder, `casual-k8s`, is used to demonstate flags and workflow.

## casual-k8s init

This creates the initial structure for casual-k8s to run.

```
$ casual-k8s init
Creating intial structure...DONE
Bootstrapping a controller...DONE
Casual K8S is ready!

Try `casual-k8s deploy core` to get started
Consider using `casual-k8s pause` to suspend
When you're done, `casual-k8s destroy` will clean up
```

## casual-k8s deploy <topology> [NAME]

This will piece together the VMs required, tag and label them as appropriate, and execute the requested
deployment against those machines. If a `NAME` for the deployment is not supplied, one will be auto-generated
based on the [petnames](https://github.com/dustinkirkland/petname) library.

If `<topology>` is a yaml bundle, that will be deployed instead.

```
$ casual-k8s deploy core
Starting kubernetes-core deployment: mutual-monster
```

```
$ casual-k8s deploy kubernetes teenyk8s
Starting kubernetes deployment: teenyk8s
```

## casual-k8s pause --all [NAME]

This will either pause all named deployments and the controller or simply pause a given, named, deployment.

```
$ casual-k8s init -q
$ casual-k8s deploy core
Starting kubernetes-core deployment: fine-flounder
$ casual-k8s pause fine-flounder
Suspending: fine-flounder...DONE

To resume run `casual-k8s resume fine-flounder`
```

```
$ casual-k8s init -q
$ casual-k8s deploy core
Starting kubernetes-core deployment: smashing-starling
$ casual-k8s pause --all
Suspending: smashing-starling...DONE
Suspending: controller...DONE

To resume run `casual-k8s resume`
```

## casual-k8s resume [NAME]

This will resume either the named deployment if it's been paused, or all paused deployments if no name
is provided.

## casual-k8s destroy [NAME]

Destroy a named deployment, keeping the controller active. If no name is provided destroy all deployments
and the controller.
