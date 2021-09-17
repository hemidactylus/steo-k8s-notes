# Notes from "Kubernetes deep dive" course

### References

Repo is here (hardly useful without watching the course but still as reference
material for manifests, etc):
[https://github.com/ACloudGuru-Resources/Course_Kubernetes_Deep_Dive_NP/](https://github.com/ACloudGuru-Resources/Course_Kubernetes_Deep_Dive_NP/)

Course is [here](https://acloudguru.com/course/kubernetes-deep-dive).

## Notes (section by section)

### Section 3 - app architecture

[Sample app here](https://github.com/nigelpoulton/k8s-sample-apps/tree/master/mysql-wordpress-pd)

### Section 4 - networking

[Demo](https://github.com/ACloudGuru-Resources/Course_Kubernetes_Deep_Dive_NP/tree/master/lesson-networking)

Need a 3-node k8s cluster.

1. ping between pods:
```
        kubectl apply -f ping-deploy.yml
        kubectl get deploy    # to check READY 3/3
        kubectl get pods -o wide

        kubectl get nodes -o jsonpath='{.items[*].spec.podCIDR}'

        # Go to a pod
        kubectl exec -it pingtest-84646bf5d4-nxs2l -- bash
        # install stuff on the pod (which is a linux barebones image)
        apt-get update
        apt-get install iputils-ping curl dnsutils iproute2 -y
        # finally:
        ping <other pod's IP>
```

2. services
```
        kubectl apply -f simple-web.yml

        # Go to a pod's bash and
        curl hello-svc:8080
        curl <clusterIP>:8080

        # Leave the pod's bash and stay on the machine 'with kubectl'
        curl <a node's IP>:30001

        # all return a sample web page
```

3. LB (supposed to work only if on an actual cloud provider)
```
        kubectl apply -f lb.yml

        # then to keep watch on command output
        kubectl get svc --watch

        # once finished the svc would have a public IP. Open a browser to:
        # <public IP>:8080
```

### Section 5 - storage

[CSI specs](https://github.com/container-storage-interface/spec)

[Storage classes, example Yaml](https://github.com/ACloudGuru-Resources/Course_Kubernetes_Deep_Dive_NP/blob/master/lesson-storage/sc.yml)

[Demo code](https://github.com/ACloudGuru-Resources/Course_Kubernetes_Deep_Dive_NP/blob/master/sample-app/mysql-wordpress-pd/mysql-deployment.yaml) - _note: does not work in PWK where there's no default storage class, must do in minikube or cloud solutions or similar._

### Section6 - from code to Kubernetes

[source code to containerize](https://github.com/ACloudGuru-Resources/Course_Kubernetes_Deep_Dive_NP/tree/master/code-k8s)

[deploy (and load balancer / nodeport) manifests](https://github.com/ACloudGuru-Resources/Course_Kubernetes_Deep_Dive_NP/tree/master/lesson-code-k8s)

**Note**: Minikube-specific trick to access the service from actual machine (minikube runs on a VM within local machine):
`minikube service --url web-nodeport`, which returns the right thing to ping (which correctly is <ip>:31000).

### Section 7 - deployments

[Example during theory](https://github.com/ACloudGuru-Resources/Course_Kubernetes_Deep_Dive_NP/blob/master/lesson-deployment/deployment.yml)

[Demo reference folder](https://github.com/ACloudGuru-Resources/Course_Kubernetes_Deep_Dive_NP/tree/master/lesson-deployment)

Demo commands, using PWK:

Created the cluster specifying version with

    kubeadm init ... --kubernetes-version=1.11.2

Check which version:

    kubectl version -o yaml

Look for ServerVersion -> GitVersion (here v1.11.2).
Grab deploy.yml (copy it into the machine, whatever)
Now apply the manifest!

    kubectl apply -f deploy.yml
    # creates deploy "test"

And immediately after check the deploy with a watch on the output

    kubectl get deploy test --watch

This watches the progression of containers being created.

Also you see a lot of info, num of replicas, image, etc with:

    kubectl describe deploy test
    
Can also look at

    kubectl get rs
    # see there is our replica set with name = 'testBLABLABLA'

Now we do a scaling operation and change spec.replicas 3 -> 6:
save the yaml and re-post it to the kube-api (`kubectl apply ...`)

Now again with `kubectl get deploy test --watch`,
you'll see AVAILABLE jump to 6 in a little while.

Now we trigger a rolling update:
we change the image `'nginx:1.12'` -> `'nginx:1.13'`, save & repeat the
`apply -f`,
and then keep track with this command:

    kubectl rollout status deploy test

(this takes a few seconds, and you see one by one the containers being replaced).
At the end (_"...successfully rolled out"_) you see the replica sets with
    
    kubectl get rs -o wide

You see TWO replica sets, one has zero replicas, the old one - that stays
around for easy rollback (just in case).

Now another rolling update, `"nginx:1.13"` -> `"nginx:1.14"`
but with

    maxSurge: 50%

(i.e. for us 3 nodes at a time).
It means "percentage of the desired number of pods". And it's rounded up to an integer.

Apply again with the record flag
(which stores the launched command in some annotation to the object itself)
    
    kubectl apply -f deploy.yml --record

So now
    
    watch kubectl get rs

and you'll see the number changing like
(time flows to the right in this progression)

    newest  0   3   3   6   6
    old     6   6   3   3   0
    older   0   0   0   0   0

There is also a history option for kubectl rollout:

    kubectl rollout history deploy test

which prints all steps (and the `--recorded` one also with the launched command)

Also:

    kubectl rollout history deploy test --revision=3

shows us all details of the third step in the history (the last one),
incl. nginx 1.14 version, the annotation etc etc.

Rolling back:

    kubectl rollout undo deploy test

This would do the trick (by default goes back to the version immediately before current)
But it is **not declaratively consistent** (now yaml != current status on cluster)
The cool way to do a rollback is **declarative** for the greater good

### Section 8 - autoscaling

#### HPA Demo (horizontal pod autoscaler)

**Minikube Note** should enable metrics server addon in minikube otherwise
it's "unknown/50%" forever in the output below.
See e.g. [this](https://stackoverflow.com/questions/44736101/kubectl-get-hpa-targetsunknow:)
Should then, after installing, check with `minikube addons list`
and enable it with `minikube addons enable metrics-server`.
I could not make this work and abandoned the hope with Minikube.

**I saw this work in the AWS EKS cluster only. See [here](eks.md) for details.**

_Note: `'namespaces'` are used here - but don't expect them to act as strong
security boundaries, they are not: you can use them for bookkeeping,
to keep dev/test/prod separate or whatever, but not different customers or
anything: that would require separate clusters!_

A svc in front of a deploy of some www-something.
A hpa on the deploy, saying: scale up when cpu > 50% of requested.
Then we shoot much load at the pod (which has a cpu-intensive something),
and we see the magic happening.

_Note: there may be, in the cluster, autoscaling enabled on cluster level.
We might see new nodes appear. Don't worry, for now it's a side issue we
can postpone (see later, "cluster autoscaler" resource)._

Demo is loosely based on
[this starting point](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/)
(but made here more declarative).

See `hpademo.yml`
[here](https://github.com/ACloudGuru-Resources/Course_Kubernetes_Deep_Dive_NP/blob/master/lesson-auto-scaling/hpademo.yml):
      
The manifest has:

- (a namespace, meh. Whose name figures in all other objects' "metadata.namespace")
- a svc (loadbalancer) (not strictly needed but let's make it a bit realistic)
- a deployment with "resource.requests.cpu: 0.2", starts with 1 replica.
  _(And its image is that of the kubernetes.io example. A simple web server that does some CPU calculations each time it's hit)_
- a HPA with `targetCPU...: 50`, trained on the deployment (through labels).

First, deploy stuff

    kubectl apply -f hpademo.yml

Then look at the status of things (takes some minutes)

    kubectl get deploy --namespace acg-ns
which is this is how you specify a namespace in the kubectl commandline (defaults to 'default').
Output at some point is `AVAILABLE = 1`.
Also look at the HPA itself:

    kubectl get hpa --namespace acg-ns

The output also says something like "TARGETS: 0%/50%" or something; also says current #replicas).

After this is up and running, spin up a stupid new pod just to issue pings (wget's actually) from it:

    kubectl run -i --tty loader --image=busybox /bin/sh
and in this shell:
    
    while true; do wget -q -O- http://acg-lb.acg-ns.svc.cluster.local; done

(Note: address = `loadBal name + namespace + default part`).

While the requests pour in, let's keep an eye on how the hpa is doing:

    kubectl get hpa --namespace acg-ns

the percentage is like "415%/50%", yay! Let's put a watch on it
    
    kubectl get hpa --namespace acg-ns --watch

and we look at the output ... replica count will soon GO UP, it works: also

    kubectl get deploy --namespace acg-ns

would give a higher number of pods for the deployment!

Let's dump the DEPLOYment yaml (on screen) with

    kubectl get deploy --namespace acg-ns -o yaml

It will have a 'status' with current status (which is dynamic) but also the specs will have `'replicas' > 1`.
(That goes to show that the HPA really goes and modifies the deploy specs (which, in turn, works on pods ... etc etc)).

Now stop the loop in the shell that was stressing it. `'get hpa'` shows "0%/50% CPU" again,
and we will see the replicas go back to the original value of 1 a bit at a time!

#### CA Demo (cluster autoscaler)

The demo in the course is done in GCP.

In the cloud provider a kube cluster is created, backed by a "node pool"
(a set of like nodes that can expand and shrink between a max and min, with a starting #nodes).

A specific GCP command to launch (given in the GCP UI) ... which just sets your kubectl config, so no big deal.
Will have sibling commands with equivalent effect on other cloud vendors.

Now you need a deployment, see [this](https://github.com/ACloudGuru-Resources/Course_Kubernetes_Deep_Dive_NP/blob/master/lesson-auto-scaling/autoscale.yml).

It's a deployment with `image=nginx`, `replicas=1` and nodes with each 1 cpu:

    spec.template.spec.containers[0].resources.requests.cpu: 0.5

It is this 0.5 that counts for the cluster autoscaler.

Now deploy

    kubectl apply -f autoscale.yml

then check a few things:
```
kubectl get deploy
# shows desired 1, available 0. How come?
# Lack of resources...? let's check

kubectl get pods
# shows pending for that one pod. Why?

kubectl describe pods
# gives a "insufficient cpu" warning of type 'failed scheduling'
# indeed you can see a new node being started! With:
kubectl get nodes
# see one ready and the new one (age=few seconds and NotReady status)

# in a little while the new node is -> Ready
# so now 'get deploy' finally shows the deploy has available=1, desired=1

# for fun, make it 10 replicas (editing the manifest)
# re-apply
kubectl get deploy # you see desired=10, available=1

# Will new nodes be created?
kubectl get nodes --watch
# in a few seconds a lot more appear! and you see them in the cloud provider's k8s service's cluster nodes view
#   (at first as 'being created' or something)
```

### Section 9 - RBAC and admission control

**Demo Note**: namespace seems to be `acg` and `acg-ns` inconsistently,
I had to change the `rolebinding.yml` at the end,
and also the last `kubectl get pods --namespace acg[-ns]` command at the end,
to see the whole thing work.

[Reference](https://github.com/ACloudGuru-Resources/Course_Kubernetes_Deep_Dive_NP/tree/master/lesson-rbac)

We configure kubectl to talk to a cluster as a new user with client certs
then we test, then we do role & rolebindings. (no admission control stuff)

**Warning**: in the demo, cert creation is not securely done with best practices - not main scope).

Demo is done in AWS with 'kops' (the k8s operations tools). Other platforms, may be different
and on managed such as EKS you may even not be able to peek so deep. That's OK, we are here to
play with roles & rolebindings, not the clusters' CAs.

we have namespaces (which is important for RBAC and best practice).
We create a client cert for user 'mia' in the 'acg' user group.
then with kubectl we try to operate as that user ... and will FAIL!
Indeed, we must still create role/rolebinding, and this time it'll work.

    kubectl get ns # shows nice namespaces

#### OpenSSL parenthesis for cert creation

(the hacky part, not even part of Kubernetes properly).

Ppenssl to create certs:

    openssl genrsa -out mia.key 2048  # creates mia.key, privkey for user

Create the CSR
_(note the CN and the O of the CSR! very important, it is how k8s later will identify the user bearing this cert!)_

    openssl req -new -key mia.key -out mia.csr -subj "/CN=mia/O=acg"  # create the CSR for Mia

I need the cert to be signed. This depends on the provider/setup. On AWS you find the CA private key (!)
in a place such as `"s3/buckets/<cluster-name-stuff>/pki/private/ca"` (it's `ca.crt`, `ca.key`, `ca.srl`)

**Minikube Note**: the CA certs are in `~/.minikube`, see [this](https://stackoverflow.com/questions/55378877/minikube-path-to-root-certificate)
and [this](https://stackoverflow.com/questions/42646109/how-i-can-add-root-ca-to-minikube?rq=1)

    openssl x509 -req -in mia.csr \
      -CA .../ca.crt \
      -CAkey .../ca.key
      -CAcreateserial -out mia.crt -days 365

Once signed, you should have a `mia.crt` file. Yay!

#### Back to the main flow:

Now for the k8s part. Kubectl uses `~/.kube/config` file, listing profiles and
users and so on for connecting to api server. We set stuff in this config with

    kubectl config set-credentials mia \
      --client-certificate=.../mia.crt \
      --client-key=.../mia.key

Then we create a context associated with this 'mia'
(the 'mia' above is a nickname, the real username is in the CN of the CSR & of the cert):

    kubectl config set-context mia \
      --cluster=acg.k8s.local --namespace=acg --user=mia

Now we connect to cluster. First,

    kubectl get pods # works so far as it is still using the usual user

Now we switch for real with
    
    kubectl config use-context mia

and:

    kubectl get pods # FAILS! (mia has no power to do that)

We still need role/rolebinding. See the files in the repo linked above. We use
a ClusterRole (just so we could use it in various namespaces, no problem).

We want to give it power of get/list/watch pods; role named 'acgrbac'

Note: the role manifest could have had

    ...
    rules:
    - apiGroups: ["", "apps"]
      resources: ["deployments", "replicasets", "pods"]
      verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]

and with `"*"` you can denote ALL API groups, ALL resources, ALL verbs.

So now
  
    kubectl apply -f yaml/role.yml
    # this would fail because I'm still mia! Must switch to the 'admin context':
    #
    kubectl apply -f yaml/role/yml --context=acg.k8s.local
    # one-off, faster than switching context you just specify it inline

a RoleBinding, associating this role with mia. the RoleBinding manifest has:

    ...
    metadata:
      name: acgrbac
      namespace: acg
    subjects:
    - kind: User
      name: mia
      apiGroup: ""
    roleRef:
      kind: ClusterRole
      name: acgrbac
      apiGroup: ""

(note it specifies a namespace: it's going to work only limited to it).
Apply the RoleBinding manifest:

    kubectl apply -f yaml/rolebinding.yml --context=acg.k8s.local

And finally can check that:

    kubectl config current-context    # to check that we are 'mia'
    kubectl get pods --namespace=acg  # it works!
    kubectl get svc --namespace=acg   # FAILS (correctly, least privilege)
