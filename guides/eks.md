# EKS

eksctl.txt

My experiences with 'eksctl' for some of the demos / real stuff testing

====

maybe it's enough to follow this basic thing for the time being
    https://docs.aws.amazon.com/eks/latest/userguide/getting-started-eksctl.html

after checking that
  aws sts get-caller-identity   # note: sts = security token service
returns
    {
        "UserId": "AIDAQYJUK62XAUWEGFO7V",
        "Account": "052186248878",
        "Arn": "arn:aws:iam::052186248878:user/stefano.lottini"
    }

Doing instructions from the above link:
  eksctl create cluster --name steocluster --region us-east-1 --fargate
  # a long output that, as expected, creates/provisions a CF stack for several minutes...
  # inspect the CF stack, it's a *lot* of resources, like 27 things (roles, control plane, sec groups, VPC/NAT stuff, ...)
  # then creating the nodegroup and a lot of other stuff
  # after 34 minutes!!!!! has finished and
    kubectl get nodes
    kubectl get svc
  # work!
  # Full output of "eksctl create cluster ..." follows:
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
        2021-08-26 23:17:36 [ℹ]  waiting for CloudFormation stack "eksctl-steocluster-cluster"
        2021-08-26 23:18:36 [ℹ]  waiting for CloudFormation stack "eksctl-steocluster-cluster"
        2021-08-26 23:19:37 [ℹ]  waiting for CloudFormation stack "eksctl-steocluster-cluster"
        2021-08-26 23:20:37 [ℹ]  waiting for CloudFormation stack "eksctl-steocluster-cluster"
        2021-08-26 23:21:38 [ℹ]  waiting for CloudFormation stack "eksctl-steocluster-cluster"
        2021-08-26 23:22:38 [ℹ]  waiting for CloudFormation stack "eksctl-steocluster-cluster"
        2021-08-26 23:23:39 [ℹ]  waiting for CloudFormation stack "eksctl-steocluster-cluster"
        2021-08-26 23:24:40 [ℹ]  waiting for CloudFormation stack "eksctl-steocluster-cluster"
        2021-08-26 23:25:40 [ℹ]  waiting for CloudFormation stack "eksctl-steocluster-cluster"
        2021-08-26 23:26:41 [ℹ]  waiting for CloudFormation stack "eksctl-steocluster-cluster"
        2021-08-26 23:27:41 [ℹ]  waiting for CloudFormation stack "eksctl-steocluster-cluster"
        2021-08-26 23:28:42 [ℹ]  waiting for CloudFormation stack "eksctl-steocluster-cluster"
        2021-08-26 23:29:42 [ℹ]  waiting for CloudFormation stack "eksctl-steocluster-cluster"
        2021-08-26 23:30:43 [ℹ]  waiting for CloudFormation stack "eksctl-steocluster-cluster"
        2021-08-26 23:32:46 [ℹ]  creating Fargate profile "fp-default" on EKS cluster "steocluster"
        2021-08-26 23:37:05 [ℹ]  created Fargate profile "fp-default" on EKS cluster "steocluster"
        2021-08-26 23:39:37 [ℹ]  "coredns" is now schedulable onto Fargate
        2021-08-26 23:41:45 [ℹ]  "coredns" is now scheduled onto Fargate
        2021-08-26 23:41:46 [ℹ]  "coredns" pods are now scheduled onto Fargate
        2021-08-26 23:43:47 [ℹ]  building managed nodegroup stack "eksctl-steocluster-nodegroup-ng-8b451620"
        2021-08-26 23:43:48 [ℹ]  deploying stack "eksctl-steocluster-nodegroup-ng-8b451620"
        2021-08-26 23:43:48 [ℹ]  waiting for CloudFormation stack "eksctl-steocluster-nodegroup-ng-8b451620"
        2021-08-26 23:44:05 [ℹ]  waiting for CloudFormation stack "eksctl-steocluster-nodegroup-ng-8b451620"
        2021-08-26 23:44:22 [ℹ]  waiting for CloudFormation stack "eksctl-steocluster-nodegroup-ng-8b451620"
        2021-08-26 23:44:42 [ℹ]  waiting for CloudFormation stack "eksctl-steocluster-nodegroup-ng-8b451620"
        2021-08-26 23:44:59 [ℹ]  waiting for CloudFormation stack "eksctl-steocluster-nodegroup-ng-8b451620"
        2021-08-26 23:45:20 [ℹ]  waiting for CloudFormation stack "eksctl-steocluster-nodegroup-ng-8b451620"
        2021-08-26 23:45:39 [ℹ]  waiting for CloudFormation stack "eksctl-steocluster-nodegroup-ng-8b451620"
        2021-08-26 23:45:59 [ℹ]  waiting for CloudFormation stack "eksctl-steocluster-nodegroup-ng-8b451620"
        2021-08-26 23:46:16 [ℹ]  waiting for CloudFormation stack "eksctl-steocluster-nodegroup-ng-8b451620"
        2021-08-26 23:46:34 [ℹ]  waiting for CloudFormation stack "eksctl-steocluster-nodegroup-ng-8b451620"
        2021-08-26 23:46:51 [ℹ]  waiting for CloudFormation stack "eksctl-steocluster-nodegroup-ng-8b451620"
        2021-08-26 23:47:08 [ℹ]  waiting for CloudFormation stack "eksctl-steocluster-nodegroup-ng-8b451620"
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

  # We try demo of HPA from section 8, 'autoscaling'
    Ok apply -f hpademo.yml
    BUT: according to this, https://docs.aws.amazon.com/eks/latest/userguide/metrics-server.html,
      (and this on HPA https://docs.aws.amazon.com/eks/latest/userguide/horizontal-pod-autoscaler.html)
      the metrics server is not automatically installed in the cluster.
      We install it with (found in the docs above):
        kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
        (check with:
          kubectl get deployment metrics-server -n kube-system
        )
      Installing it takes about one minute

    If you log in as the IAM user corresponding to that of 'aws CLI', you see all resources
    including the node group in the cluster!

    Ok, back on track with the metrics working. Now the
      kubectl get hpa --namespace acg-ns
    returns a 0%/50% in targets, yay!

    (by the way, the service deployed in the example - a web server that eats CPU to return 'OK!' - has an external domain, all handled within aws by the deployment!)

    ??? For some reason in the 'busybox' image, supposedly identical to deployed e.g. on minikube,
    there is no "/bin/bash" but then if I deploy the pod and then exec 'sh' on it, I can run the loop with the wget.
    ???

    All proceeds as in the demo, with the dumped deployment yaml as follows:
      kubectl get deploy --namespace acg-ns -o yaml
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

  # We now try the CA demo - maybe requires some special things (the aws-specific version of them)
  # to create CA in aws, you do some tricks with ... you guessed it: roles and permission and IAM black magic.
  # According to this: https://docs.aws.amazon.com/eks/latest/userguide/cluster-autoscaler.html,
  # since we created the cluster with eksctl, it should be quick. Let's see:
    Now, the doc say to invoke a command "eksctl create iamserviceaccount ..." before installing the CA.
    The doc is unclear on what to put as a policy when you DID NOT use the --asg-access flag back in creating the node group
    (ahem, the cluster really: the node group was automatically created by eksctl in the
    giant 34-minutes cloudformation festival)
    So by looking here:
      https://docs.aws.amazon.com/eks/latest/userguide/create-node-role.html
    it may be that the right thing is to look for a "ROLE" and alter the command to attach a role and not a policy, as in:
      eksctl create iamserviceaccount   --cluster=steocluster   --namespace=kube-system   --name=cluster-autoscaler   --override-existing-serviceaccounts   --approve --attach-role-arn arn:aws:iam::052186248878:role/eksctl-steocluster-nodegroup-ng-8-NodeInstanceRole-SL63QZD8GAFC
    (role found in IAM as suggested in the create-node-role.html page).
    But this tries and gives an error, "no IAM OIDC provider associated with cluster ... blabla. Try <command>"
    where <command> is:
      eksctl utils associate-iam-oidc-provider --region=us-east-1 --cluster=steocluster
    you launch it with '--approve' added and should create an "IAM Open ID Connect provider" associated to the k8s cluster:
    And it did!
      eksctl utils associate-iam-oidc-provider --region=us-east-1 --cluster=steocluster --approve
          2021-08-27 01:05:53 [ℹ]  eksctl version 0.62.0
          2021-08-27 01:05:53 [ℹ]  using region us-east-1
          2021-08-27 01:05:54 [ℹ]  will create IAM Open ID Connect provider for cluster "steocluster" in "us-east-1"
          2021-08-27 01:05:55 [✔]  created IAM Open ID Connect provider for cluster "steocluster" in "us-east-1"
    So now the above command (create iamserviceaccount ...) worked!
      eksctl create iamserviceaccount   --cluster=steocluster   --namespace=kube-system   --name=cluster-autoscaler   --override-existing-serviceaccounts   --approve --attach-role-arn arn:aws:iam::052186248878:role/eksctl-steocluster-nodegroup-ng-8-NodeInstanceRole-SL63QZD8GAFC
          2021-08-27 01:06:17 [ℹ]  eksctl version 0.62.0
          2021-08-27 01:06:17 [ℹ]  using region us-east-1
          2021-08-27 01:06:19 [ℹ]  1 iamserviceaccount (kube-system/cluster-autoscaler) was included (based on the include/exclude rules)
          2021-08-27 01:06:19 [!]  metadata of serviceaccounts that exist in Kubernetes will be updated, as --override-existing-serviceaccounts was set
          2021-08-27 01:06:19 [ℹ]  1 task: { create serviceaccount "kube-system/cluster-autoscaler" }
          2021-08-27 01:06:19 [ℹ]  created serviceaccount "kube-system/cluster-autoscaler"
    Now we are back to following the "cluster-autoscaler.html" page in the doc maze.
    (step 1 of Deploy) We can kubectl apply the CA in the cluster:
      kubectl apply -f https://raw.githubusercontent.com/kubernetes/autoscaler/master/cluster-autoscaler/cloudprovider/aws/examples/cluster-autoscaler-autodiscover.yaml
      # (gives a warning: missing some annotations so it's not entirely declarative, autopatched for now but please)
    (step 2) annotate the cluster-autoscaler service account, giving the ARN of the "role created previously" (in the docs so far it has spoken of a policy, not a role - although it was a role I used and attached to the iamserviceaccount. Bah)
      kubectl annotate serviceaccount cluster-autoscaler   -n kube-system   eks.amazonaws.com/role-arn=arn:aws:iam::052186248878:role/eksctl-steocluster-nodegroup-ng-8-NodeInstanceRole-SL63QZD8GAFC
      Step 2 gives an ERROR = there is already such an annotation
      (looking closely, the contents are already what we wanted to annotate here, same role, so maybe we can skip this?)
    (step 3) 
      patch the deployment to add a 'safe-to-evict: false' annotation to the CA pods (?)
        kubectl patch deployment cluster-autoscaler   -n kube-system   -p '{"spec":{"template":{"metadata":{"annotations":{"cluster-autoscaler.kubernetes.io/safe-to-evict": "false"}}}}}'
            deployment.apps/cluster-autoscaler patched
    (step 4)
      edit the CA deployment's manifest
        - replace <YOUR CLUSTER NAME> with 'steocluster' (in TWO points)
        and other options at the end of in 'spec.containers.command':
          --balance-similar-node-groups
          --skip-nodes-with-system-pods=false

    (step 5 open the CA releases page and look for the latest CA matching my k8s version.
    page: https://github.com/kubernetes/autoscaler/releases
    for k8s 1.20 -> it is 1.20.0)

    (step 6) set the CA image tag to the version just found:
      kubectl set image deployment cluster-autoscaler   -n kube-system   cluster-autoscaler=k8s.gcr.io/autoscaling/cluster-autoscaler:v1.20.0
          deployment.apps/cluster-autoscaler image updated
      (doc is inaccurate, does not mention that you must replace "<1.21.n>" including the "<", ">"


    TEST
      kubectl -n kube-system logs -f deployment.apps/cluster-autoscaler
      ** seems to do "something"

  # Let us assume the CA is installed (bah) and try he demo.
      kubectl apply -f autoscale.yml
      kubectl get deploy    # shows available 0/1
      kubectl get pods      # shows one pod pending -> then running
      # so as in the demo, also get deploy shows 1/1 running all right.

      # we edit the yaml, "spec.replicas -> 6", re-apply and
        watch the node count, 'kubectl get nodes'
        and the 'kubectl get deploy'

        * The get nodes indeed is growing like crazy, and also the get deploy!
        Deploy reaches 6/6!
        And nodes has added 5 fargate instances! (indeed replicas went 1->6)
        WOW.

        Also same increase in fargates can be seen in the EKS console for the cluster. Yay.

# DELETING THE WHOLE CLUSTER
Took only 8 minutes!

      eksctl delete cluster steocluster
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
            2021-08-27 01:36:28 [ℹ]  waiting for CloudFormation stack "eksctl-steocluster-nodegroup-ng-8b451620"
            2021-08-27 01:36:45 [ℹ]  waiting for CloudFormation stack "eksctl-steocluster-nodegroup-ng-8b451620"
            2021-08-27 01:37:06 [ℹ]  waiting for CloudFormation stack "eksctl-steocluster-nodegroup-ng-8b451620"
            2021-08-27 01:37:23 [ℹ]  waiting for CloudFormation stack "eksctl-steocluster-nodegroup-ng-8b451620"
            2021-08-27 01:37:43 [ℹ]  waiting for CloudFormation stack "eksctl-steocluster-nodegroup-ng-8b451620"
            2021-08-27 01:38:03 [ℹ]  waiting for CloudFormation stack "eksctl-steocluster-nodegroup-ng-8b451620"
            2021-08-27 01:38:22 [ℹ]  waiting for CloudFormation stack "eksctl-steocluster-nodegroup-ng-8b451620"
            2021-08-27 01:38:39 [ℹ]  waiting for CloudFormation stack "eksctl-steocluster-nodegroup-ng-8b451620"
            2021-08-27 01:38:58 [ℹ]  waiting for CloudFormation stack "eksctl-steocluster-nodegroup-ng-8b451620"
            2021-08-27 01:39:15 [ℹ]  waiting for CloudFormation stack "eksctl-steocluster-nodegroup-ng-8b451620"
            2021-08-27 01:39:31 [ℹ]  waiting for CloudFormation stack "eksctl-steocluster-nodegroup-ng-8b451620"
            2021-08-27 01:39:50 [ℹ]  waiting for CloudFormation stack "eksctl-steocluster-nodegroup-ng-8b451620"
            2021-08-27 01:39:52 [ℹ]  will delete stack "eksctl-steocluster-cluster"
            2021-08-27 01:39:52 [✔]  all cluster resources were deleted
