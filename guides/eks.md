# AWS EKS Survival Guide

My goal was: let us just have a vanilla K8s cluster up and running in EKS
to be able to finally test HPA and possibly CA.

As usual with AWS, you end up in a jungle of IAM roles and permissions
even for the simplest thing, with a matrioska structure of doc pages
instructing you to create this and that IAM roles, etc etc.

I soon abandoned the idea of creating everything by hand (too many doc
page tabs open) and went for the "easy" way.

The easy way is using `eksctl`, a command-line utility from AWS, companion
(and which makes use of) the `aws` CLI, which does a lot of things for you.

Still, creation of the cluster takes _quite some time_: behind the scenes, a lot of
rather heavy operations are executed, as CloudFormation stacks, which
involve creation of EC2 instances, VPCs, LBs, ... a lot of moving parts.

### Preliminaries

Have `aws` CLI installed and set up on your machine and check who you are with

    aws sts get-caller-identity   # note: sts = security token service
    #
    # should return something like:
    #     {
    #         "UserId": "AID...",
    #         "Account": "05...",
    #         "Arn": "arn:aws:iam::05...:user/XXXYYYZZZ"
    #     }

What follows has been done as described in the doc
[here](https://docs.aws.amazon.com/eks/latest/userguide/getting-started-eksctl.html).

Now it's time to install `eksctl`, see the linked doc.

### Creation of the K8s cluster

I'm calling my cluster `steocluster`. Start with:

    eksctl create cluster --name steocluster --region us-east-1 --fargate

A very long output follows. A CloudFormation stack is created/provisioned.
The thing lasts several minutes: if you inspect the CF stack you see that
it's a _lot_ of resources (I counted 27: roles, control plane,
security groups, VPC/NAT things; creation of the NodeGroup, etc etc).
The (abridged) output follows:

        2021-08-26 23:16:34 [ℹ]  eksctl version 0.62.0
        2021-08-26 23:16:34 [ℹ]  using region us-east-1
        2021-08-26 23:16:34 [ℹ]  setting availability zones to [us-east-1f us-east-1b]
        2021-08-26 23:16:34 [ℹ]  subnets for us-east-1f - public:192.168.0.0/19 private:192.168.64.0/19
        2021-08-26 23:16:34 [ℹ]  subnets for us-east-1b - public:192.168.32.0/19 private:192.168.96.0/19
        2021-08-26 23:16:34 [ℹ]  nodegroup "ng-8b451620" will use "" [AmazonLinux2/1.20]
        2021-08-26 23:16:34 [ℹ]  using Kubernetes version 1.20
        2021-08-26 23:16:34 [ℹ]  creating EKS cluster "steocluster" in "us-east-1" region with Fargate profile and managed nodes
        2021-08-26 23:16:34 [ℹ]  will create 2 separate CloudFormation stacks for cluster itself and the initial managed nodegroup
        2021-08-26 23:16:34 [ℹ]  if you encounter any issues, check CloudFormation console or try 'eksctl utils describe-stacks --region=us-east-1 --cluster=steocluster'
        2021-08-26 23:16:34 [ℹ]  CloudWatch logging will not be enabled for cluster "steocluster" in "us-east-1"
        2021-08-26 23:16:34 [ℹ]  you can enable it with 'eksctl utils update-cluster-logging --enable-types={SPECIFY-YOUR-LOG-TYPES-HERE (e.g. all)} --region=us-east-1 --cluster=steocluster'
        2021-08-26 23:16:34 [ℹ]  Kubernetes API endpoint access will use default of {publicAccess=true, privateAccess=false} for cluster "steocluster" in "us-east-1"
        2021-08-26 23:16:34 [ℹ]  2 sequential tasks: { create cluster control plane "steocluster", 3 sequential sub-tasks: { 2 sequential sub-tasks: { wait for control plane to become ready, create fargate profiles }, 1 task: { create addons }, create managed nodegroup "ng-8b451620" } }
        2021-08-26 23:16:34 [ℹ]  building cluster stack "eksctl-steocluster-cluster"
        2021-08-26 23:16:35 [ℹ]  deploying stack "eksctl-steocluster-cluster"
        2021-08-26 23:17:05 [ℹ]  waiting for CloudFormation stack "eksctl-steocluster-cluster"
        [...]
        2021-08-26 23:32:46 [ℹ]  creating Fargate profile "fp-default" on EKS cluster "steocluster"
        2021-08-26 23:37:05 [ℹ]  created Fargate profile "fp-default" on EKS cluster "steocluster"
        2021-08-26 23:39:37 [ℹ]  "coredns" is now schedulable onto Fargate
        2021-08-26 23:41:45 [ℹ]  "coredns" is now scheduled onto Fargate
        2021-08-26 23:41:46 [ℹ]  "coredns" pods are now scheduled onto Fargate
        2021-08-26 23:43:47 [ℹ]  building managed nodegroup stack "eksctl-steocluster-nodegroup-ng-8b451620"
        2021-08-26 23:43:48 [ℹ]  deploying stack "eksctl-steocluster-nodegroup-ng-8b451620"
        2021-08-26 23:43:48 [ℹ]  waiting for CloudFormation stack "eksctl-steocluster-nodegroup-ng-8b451620"
        [...]
        2021-08-26 23:47:08 [ℹ]  waiting for the control plane availability...
        2021-08-26 23:47:08 [✔]  saved kubeconfig as "/home/stefano/.kube/config"
        2021-08-26 23:47:08 [ℹ]  no tasks
        2021-08-26 23:47:08 [✔]  all EKS cluster resources for "steocluster" have been created
        2021-08-26 23:47:09 [ℹ]  nodegroup "ng-8b451620" has 2 node(s)
        2021-08-26 23:47:09 [ℹ]  node "ip-192-168-12-180.ec2.internal" is ready
        2021-08-26 23:47:09 [ℹ]  node "ip-192-168-34-210.ec2.internal" is ready
        2021-08-26 23:47:09 [ℹ]  waiting for at least 2 node(s) to become ready in "ng-8b451620"
        2021-08-26 23:47:09 [ℹ]  nodegroup "ng-8b451620" has 2 node(s)
        2021-08-26 23:47:09 [ℹ]  node "ip-192-168-12-180.ec2.internal" is ready
        2021-08-26 23:47:09 [ℹ]  node "ip-192-168-34-210.ec2.internal" is ready
        2021-08-26 23:49:14 [ℹ]  kubectl command should work with "/home/stefano/.kube/config", try 'kubectl get nodes'
        2021-08-26 23:49:14 [✔]  EKS cluster "steocluster" in "us-east-1" region is ready

After 34 minutes the cluster is ready. Your `~/.kube/config` has also been touched and you can easily test with:

    kubectl get nodes
    kubectl get svc

Moreover, if you log in to the AWS Console as the IAM user corresponding to that of the aws CLI used above,
you'll see all AWS resources pertaining to the K8s cluster, including the node group on which it resides.

### Trying the KDD demos for HPA and CA

We can finally try the autoscaling demos from the [KDD course](kdd/md).

#### HPA Demo

All right with the
    
    kubectl apply -f hpademo.yml

But _beware_: according to [this document](https://docs.aws.amazon.com/eks/latest/userguide/metrics-server.html),
the metrics server is not automatically installed in the cluster (see also [this page on HPA](https://docs.aws.amazon.com/eks/latest/userguide/horizontal-pod-autoscaler.html)).

To install the metrics server, the doc above gives the command

    kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

This takes about one minute and at the end you can check with

    kubectl get deployment metrics-server -n kube-system

Once the metrics server is working, the load test from the demo,

    kubectl get hpa --namespace acg-ns

will return, under "targets", a nice `0%/50%`.

_Note: by the way, the service deployed in the example - a web server that eats CPU on purpose to return 'OK!' -
has an external reachable domain, all handled within AWS by the deployment._

**Note** For some reason in the 'busybox' image, supposedly identical to the one deployed e.g. on minikube,
there is no `/bin/bash` but then if I deploy the pod and then exec 'sh' on it, I can run the loop with the wget.
Unclear why. It should be the very same image yet it has different shell
(related to the cluster being on AWS? unlikely).

Everything proceeds as in the demo. Here is the dumped deployment yaml from
`kubectl get deploy --namespace acg-ns -o yaml`:

```
apiVersion: v1
items:
- apiVersion: apps/v1
  kind: Deployment
  metadata:
    annotations:
      deployment.kubernetes.io/revision: "1"
      kubectl.kubernetes.io/last-applied-configuration: |
        {"apiVersion":"apps/v1","kind":"Deployment","metadata":{"annotations":{},"labels":{"app":"acg-stress"},"name":"acg-web","namespace":"acg-ns"},"spec":{"replicas":1,"selector":{"matchLabels":{"app":"acg-stress"}},"strategy":{"rollingUpdate":{"maxSurge":1,"maxUnavailable":0},"type":"RollingUpdate"},"template":{"metadata":{"labels":{"app":"acg-stress"}},"spec":{"containers":[{"image":"k8s.gcr.io/hpa-example","name":"stresser","ports":[{"containerPort":80}],"resources":{"requests":{"cpu":0.2}}}]}}}}
    creationTimestamp: "2021-08-26T22:23:07Z"
    generation: 3
    labels:
      app: acg-stress
    name: acg-web
    namespace: acg-ns
    resourceVersion: "12399"
    uid: 2e89e3cb-be9d-44a1-a83d-33ea2045e015
  spec:
    progressDeadlineSeconds: 600
    replicas: 8
    revisionHistoryLimit: 10
    selector:
      matchLabels:
        app: acg-stress
    strategy:
      rollingUpdate:
        maxSurge: 1
        maxUnavailable: 0
      type: RollingUpdate
    template:
      metadata:
        creationTimestamp: null
        labels:
          app: acg-stress
      spec:
        containers:
        - image: k8s.gcr.io/hpa-example
          imagePullPolicy: Always
          name: stresser
          ports:
          - containerPort: 80
            protocol: TCP
          resources:
            requests:
              cpu: 200m
          terminationMessagePath: /dev/termination-log
          terminationMessagePolicy: File
        dnsPolicy: ClusterFirst
        restartPolicy: Always
        schedulerName: default-scheduler
        securityContext: {}
        terminationGracePeriodSeconds: 30
  status:
    availableReplicas: 8
    conditions:
    - lastTransitionTime: "2021-08-26T22:23:07Z"
      lastUpdateTime: "2021-08-26T22:23:23Z"
      message: ReplicaSet "acg-web-6dbf94b886" has successfully progressed.
      reason: NewReplicaSetAvailable
      status: "True"
      type: Progressing
    - lastTransitionTime: "2021-08-26T22:46:27Z"
      lastUpdateTime: "2021-08-26T22:46:27Z"
      message: Deployment has minimum availability.
      reason: MinimumReplicasAvailable
      status: "True"
      type: Available
    observedGeneration: 3
    readyReplicas: 8
    replicas: 8
    updatedReplicas: 8
kind: List
metadata:
  resourceVersion: ""
  selfLink: ""
```

#### CA Demo

To have autoscaling in our EKS cluster, some more AWS-specific meddling
is required. To enable CA in AWS, indeed, you do some more tricks with...
you guessed it: roles and permissions and IAM black magic.

Reference page is [this one](https://docs.aws.amazon.com/eks/latest/userguide/cluster-autoscaler.html).
Since this cluster was created with the help of `eksctl`, it should be relatively quick. Let's see.

##### Service account creation

Now, the doc says to invoke a command `eksctl create iamserviceaccount ...` before installing the CA.

But there's something unclear: what to put as a policy, since we _did not_ use the `--asg-access` flag back when
creating the node group (or, better, the cluster: indeed the node group was automatically
created by `eksctl` in the 34-minutes CloudFormation party).

So by looking [here](https://docs.aws.amazon.com/eks/latest/userguide/create-node-role.html),
I conclude that the right thing to do should be this: look for a _role_
and alter the command so that it attaches a _role_ and not a policy.

I ended up with this:
    
    # Note: this won't work yet, see below
    eksctl create iamserviceaccount --cluster=steocluster --namespace=kube-system --name=cluster-autoscaler \
      --override-existing-serviceaccounts --approve \
      --attach-role-arn arn:aws:iam::052186248878:role/eksctl-steocluster-nodegroup-ng-8-NodeInstanceRole-SL63QZD8GAFC

where the role ARN is found in IAM as suggested in the doc page linked right above.

But **Beware**! This command gives an error, _no IAM OIDC provider associated with cluster ... blabla. Try <COMMAND>_

The suggested command is something like

    eksctl utils associate-iam-oidc-provider --region=us-east-1 --cluster=steocluster  # not complete yet!

It does not work like this but looking at how it complains I see I can try adding an `--approve` flag, as in

    eksctl utils associate-iam-oidc-provider --region=us-east-1 --cluster=steocluster --approve

which finally succeeds and creates an "IAM Open ID Connect provider" associated to the k8s cluster. Its output is:

    2021-08-27 01:05:53 [ℹ]  eksctl version 0.62.0
    2021-08-27 01:05:53 [ℹ]  using region us-east-1
    2021-08-27 01:05:54 [ℹ]  will create IAM Open ID Connect provider for cluster "steocluster" in "us-east-1"
    2021-08-27 01:05:55 [✔]  created IAM Open ID Connect provider for cluster "steocluster" in "us-east-1"

Now the `create iamserviceaccount` works:

    eksctl create iamserviceaccount --cluster=steocluster --namespace=kube-system --name=cluster-autoscaler \
      --override-existing-serviceaccounts --approve \
      --attach-role-arn arn:aws:iam::052186248878:role/eksctl-steocluster-nodegroup-ng-8-NodeInstanceRole-SL63QZD8GAFC

with output such as

    2021-08-27 01:06:17 [ℹ]  eksctl version 0.62.0
    2021-08-27 01:06:17 [ℹ]  using region us-east-1
    2021-08-27 01:06:19 [ℹ]  1 iamserviceaccount (kube-system/cluster-autoscaler) was included (based on the include/exclude rules)
    2021-08-27 01:06:19 [!]  metadata of serviceaccounts that exist in Kubernetes will be updated, as --override-existing-serviceaccounts was set
    2021-08-27 01:06:19 [ℹ]  1 task: { create serviceaccount "kube-system/cluster-autoscaler" }
    2021-08-27 01:06:19 [ℹ]  created serviceaccount "kube-system/cluster-autoscaler"

and we can get back on track with setting up the cluster autoscaler.

##### Back to installing the CA

With reference to the "Deploy" steps in  corresponding
[doc page](https://docs.aws.amazon.com/eks/latest/userguide/cluster-autoscaler.html),
also linked above, we can resume:

**Step 1 of Deploy**. We can `kubectl apply` the CA in the cluster:

    kubectl apply -f https://raw.githubusercontent.com/kubernetes/autoscaler/master/cluster-autoscaler/cloudprovider/aws/examples/cluster-autoscaler-autodiscover.yaml

It works but gives a warning to the effect of _missing some annotations so it's not entirely declarative, autopatched this time, but please!_

**Step 2**. Annotate the `cluster-autoscaler` service account, giving the ARN of the "role created previously"
(although in the docs, before this point, it was supposed to be a policy and not a role; on the other hand I used
a role indeed and attached it to the iamserviceaccount. Small inconsistency, but apparently resolved in favour of a role).

    # This won't succeed!
    kubectl annotate serviceaccount cluster-autoscaler -n kube-system eks.amazonaws.com/role-arn=arn:aws:iam::052186248878:role/eksctl-steocluster-nodegroup-ng-8-NodeInstanceRole-SL63QZD8GAFC

Step 2 gives an _Error_ ("there is already such an annotation").
Looking closely, the contents are already what we wanted to annotate here, same role, so maybe we can skip this after all.

We skip this indeed.

**Step 3**. Patch the deployment to add a `'safe-to-evict: false'` annotation to the CA pods.

    kubectl patch deployment cluster-autoscaler -n kube-system \
      -p '{"spec":{"template":{"metadata":{"annotations":{"cluster-autoscaler.kubernetes.io/safe-to-evict": "false"}}}}}'

Output is `deployment.apps/cluster-autoscaler patched`.

**Step 4**. Edit the CA deployment's manifest to tweak a few things. This is done opening a text editor with

    kubectl -n kube-system edit deployment.apps/cluster-autoscaler

and:

- replacing `<YOUR CLUSTER NAME>` with `'steocluster'`, _in TWO places_;
- adding options at the end of the list in `'spec.containers.command'`:
```
    --balance-similar-node-groups
    --skip-nodes-with-system-pods=false
```

**Step 5**. Open the [CA releases](https://github.com/kubernetes/autoscaler/releases) page
and look for the latest CA matching my k8s version. I found `1.20` for k8s, so for us here it is `1.20.0`.

**Step 6**. Set the CA image tag to the version just found:

    kubectl set image deployment cluster-autoscaler -n kube-system \
      cluster-autoscaler=k8s.gcr.io/autoscaling/cluster-autoscaler:v1.20.0

Output is `deployment.apps/cluster-autoscaler image updated`.

_Careful_, the doc leaves the `<` and `>` which have to be edited away!

**Verify the CA is deployed**.

Command

    kubectl -n kube-system logs -f deployment.apps/cluster-autoscaler

seems to do "something". Ok, installed.

##### Finally the CA demo as in [KDD](kdd.md).

First

    kubectl apply -f autoscale.yml

Then

    kubectl get deploy

which shows "AVAILABLE 0/1".

Then

    kubectl get pods

first shows one pending pod, then it becomes running. And now the `get deploy`
gives "1/1 running": all is fine.

We then edit the yaml, set `spec.replicas` to 6, re-apply and
watch the node count with `kubectl get nodes` and `kubectl get deploy`.

Both counts indeed grow like crazy!

- Output for deploys reaches 6/6;
- Output for nodes also shows that 5 new Fargate instances were added;
- also the same increase can be seen in the EKS console for this cluster.

So everything works. Quick, isn't it?

### Deletion of the whole cluster

This took only 8 minutes and is a single command (and some more patience):

    eksctl delete cluster steocluster

Here is its (abridged) output:

    2021-08-27 01:31:47 [ℹ]  eksctl version 0.62.0
    2021-08-27 01:31:47 [ℹ]  using region us-east-1
    2021-08-27 01:31:47 [ℹ]  deleting EKS cluster "steocluster"
    2021-08-27 01:31:49 [ℹ]  deleting Fargate profile "fp-default"
    2021-08-27 01:36:06 [ℹ]  deleted Fargate profile "fp-default"
    2021-08-27 01:36:06 [ℹ]  deleted 1 Fargate profile(s)
    2021-08-27 01:36:08 [✔]  kubeconfig has been updated
    2021-08-27 01:36:08 [ℹ]  cleaning up AWS load balancers created by Kubernetes objects of Kind Service or Ingress
    2021-08-27 01:36:11 [ℹ]  3 sequential tasks: { delete nodegroup "ng-8b451620", delete IAM OIDC provider, delete cluster control plane "steocluster" [async] }
    2021-08-27 01:36:12 [ℹ]  will delete stack "eksctl-steocluster-nodegroup-ng-8b451620"
    2021-08-27 01:36:12 [ℹ]  waiting for stack "eksctl-steocluster-nodegroup-ng-8b451620" to get deleted
    2021-08-27 01:36:12 [ℹ]  waiting for CloudFormation stack "eksctl-steocluster-nodegroup-ng-8b451620"
    [...]
    2021-08-27 01:39:52 [ℹ]  will delete stack "eksctl-steocluster-cluster"
    2021-08-27 01:39:52 [✔]  all cluster resources were deleted
