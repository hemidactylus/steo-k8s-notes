# Bonsai

local-running kubernetes clusters and other learning environments.

For testing, or even for serious stuff. Let's say: "for when you don't want
to wait 40 minutes for EKS to start".

## Minikube

Spins up local VM (or multiple as well) on which the k8s cluster is then
created. Handles the k8s network and so on, configures kubeconfig for
easy kubectl access.

[Install it](https://minikube.sigs.k8s.io/docs/start/) and then:

    minikube start
    # ... do stuff
    minikube stop
    minikube delete --all

Can also start multi-node local cluster, e.g.

    minikube start --nodes 2 -p multinode-demo

It's a full-fledged k8s cluster so the usual commands will work (kubectl ...)

**Warning** installing the metrics plugin (e.g. to have HPAs) is a bloody mess
I could not tame.

## KinD

More lightweight, based on Dockerized images instead of VMs.

[nstall it](https://kind.sigs.k8s.io/)
(by piping something to sudo bash) and then something like

    kind create cluster
    # ... do stuff
    kind delete cluster

Seems neater than Minikube I must say.

(No idea yet if it supports metrics for HPAs)

## PWK

a.k.a. "Play with Kubernetes". This is a Web service that offers
K8s clusters for free in the cloud, with a lifetime of 4 hours.

Go to [the site](https://labs.play-with-k8s.com/) and you can spin
up a cluster (with multiple nodes as well) and play with it in
as many faux-consoles (i.e. command-line consoles rendered in browser pages).

**Warning** Sadly, no support for metrics, so can't play with HPAs
