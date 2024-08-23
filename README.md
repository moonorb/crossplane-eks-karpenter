# crossplane-eks-karpenter
This repository contains the code to showcase how to provision EKS Cluster, 2 Helm Charts and Karpenter with IRSA. Some of the XRDs and Compositions on this repository are modified version of the [Upbound repositories](https://github.com/upbound)

Feel free to use this code to deploy EKS and Karpenter or any Helm Charts. 

### Pre-requisites: 
You must have Docker running on your host so [kind(kubernetes in Docker)](https://github.com/kubernetes-sigs/kind) can run. I am running the steps below on an Ubuntu host with 16GB memory.

### Clone the Repo
```
git clone https://github.com/moonorb/crossplane-eks-karpenter.git
cd crossplane-eks-karpenter
```

### kind(Kubernetes IN Docker)
nix shell is used to install kind as well as kubectl and a few other required software. 
```
sh <(curl -L https://nixos.org/nix/install) --no-daemon
nix-shell scripts/shell.nix --run $SHELL
```

### Update your AWS credentials file
```
cat <<EOF > credentials/aws-credentials.txt
[default]
aws_access_key_id = AKIAZQ3DPHSQIF4FB22C
aws_secret_access_key = iB5/ppehdnkLT4RMfh+NCT6zFpG8b8UGIXtRJ5xM
# These creds are fake...
EOF
```

### Run installation script
This script will install a single node Kubernetes Management cluster via kind as well as Crossplane controllers and required providers on this cluster. It takes approximately 4-5 minutes for all the pods to be ready. 
```
./scripts/install-crossplane.nix
```

### Check the pods
```
john@moonpod:~/crossplane-eks-karpenter$ kubectl get po -A
NAMESPACE            NAME                                                          READY   STATUS    RESTARTS   AGE
crossplane-system    crossplane-6b5b8f9549-6cfw5                                   1/1     Running   0          6m8s
crossplane-system    crossplane-rbac-manager-bcddfb7-nmxdx                         1/1     Running   0          6m8s
crossplane-system    function-auto-ready-ad9454a37aa7-6c66f4984d-nvdsk             1/1     Running   0          5m39s
crossplane-system    function-cel-filter-e27bffada1b4-646878f869-rcs46             1/1     Running   0          5m31s
crossplane-system    function-patch-and-transform-6a1ab24d2512-564dd584fd-qrk48    1/1     Running   0          5m31s
crossplane-system    function-sequencer-c3d57e50b393-6c8c98d4fb-sc72t              1/1     Running   0          5m38s
crossplane-system    provider-aws-cloudwatch-6944ea02bce9-df449975-qzngl           1/1     Running   0          5m9s
crossplane-system    provider-aws-cloudwatchevents-764b5433847c-66f9d798cd-f7c8l   1/1     Running   0          5m18s
crossplane-system    provider-aws-cloudwatchlogs-0443494a1790-56b559c7f8-bwgpn     1/1     Running   0          5m18s
crossplane-system    provider-aws-ec2-2c1724a70b5c-989897978-t5hq8                 1/1     Running   0          5m6s
crossplane-system    provider-aws-eks-5d74bd862e84-598dc54dd5-m2tp5                1/1     Running   0          5m16s
crossplane-system    provider-aws-iam-9f6f8ce3d288-869684576-x6r4s                 1/1     Running   0          5m14s
crossplane-system    provider-aws-sqs-3f83196106e4-55849b6464-hnqv4                1/1     Running   0          5m16s
crossplane-system    provider-helm-239d2e74884b-5f74b4bdc7-9bql8                   1/1     Running   0          5m35s
crossplane-system    provider-kubernetes-101d5a6d80d3-7858c79b58-c65hf             1/1     Running   0          5m18s
crossplane-system    upbound-provider-family-aws-503b7f0129e5-7449f64b7d-tbkgd     1/1     Running   0          5m25s
kube-system          coredns-7db6d8ff4d-dhfz9                                      1/1     Running   0          6m8s
kube-system          coredns-7db6d8ff4d-ztq5l                                      1/1     Running   0          6m8s
kube-system          etcd-kind-control-plane                                       1/1     Running   0          6m30s
kube-system          kindnet-p246z                                                 1/1     Running   0          6m8s
kube-system          kube-apiserver-kind-control-plane                             1/1     Running   0          6m31s
kube-system          kube-controller-manager-kind-control-plane                    1/1     Running   0          6m31s
kube-system          kube-proxy-gbdnf                                              1/1     Running   0          6m8s
kube-system          kube-scheduler-kind-control-plane                             1/1     Running   0          6m33s
local-path-storage   local-path-provisioner-988d74bc-kjkvm                         1/1     Running   0          6m8s

```
You are ready to deploy the resources when all the pods are in Running state. 

### Deploy EKS
Configure subnets in [EKS Claim](https://github.com/moonorb/crossplane-eks-karpenter/blob/main/examples/eks.yaml)
```
kubectl apply -f apis/eks/definition.yaml
kubectl apply -f apis/eks/composition.yaml
kubectl apply -f examples/eks.yaml
```

Confirm that all the managed resources are provisioned successfully after 15 mins. 
```
watch "crossplane beta trace ekscluster moonorb-test-x -n test-ns"
NAME                                                        SYNCED   READY   STATUS
EKSCluster/moonorb-test-x (test-ns)                         True     True    Available
└─ XEKS/moonorb-test-x-6ckdm                                True     True    Available
   ├─ SecurityGroupIngressRule/moonorb-test-x-6ckdm-rg844   True     True    Available
   ├─ SecurityGroup/moonorb-test-x-6ckdm-srpl2              True     True    Available
   ├─ Addon/moonorb-test-x-6ckdm-q7pfg                      True     True    Available
   ├─ Addon/moonorb-test-x-6ckdm-xsghc                      True     True    Available
   ├─ ClusterAuth/moonorb-test-x-6ckdm-v2d7t                True     True    Available
   ├─ Cluster/moonorb-test-x                                True     True    Available
   ├─ NodeGroup/moonorb-test-x-6ckdm-257z4                  True     True    Available
   ├─ ProviderConfig/moonorb-test-x-helm                    -        -
   ├─ OpenIDConnectProvider/moonorb-test-x-6ckdm-hknpp      True     True    Available
   ├─ RolePolicyAttachment/moonorb-test-x-6ckdm-77sq5       True     True    Available
   ├─ RolePolicyAttachment/moonorb-test-x-6ckdm-dktqp       True     True    Available
   ├─ RolePolicyAttachment/moonorb-test-x-6ckdm-gpmv6       True     True    Available
   ├─ RolePolicyAttachment/moonorb-test-x-6ckdm-rf6g7       True     True    Available
   ├─ RolePolicyAttachment/moonorb-test-x-6ckdm-zkbcf       True     True    Available
   ├─ Role/moonorb-test-x-6ckdm-6fw58                       True     True    Available
   ├─ Role/moonorb-test-x-6ckdm-s7vf2                       True     True    Available
   ├─ Object/moonorb-test-x-aws-auth                        True     True    Available
   ├─ Object/moonorb-test-x-irsa-settings                   True     True    Available
   └─ ProviderConfig/moonorb-test-x                         -        -
```

### Deploy IRSA
```
kubectl apply -f apis/irsa/definition.yaml 
kubectl apply -f apis/irsa/composition.yaml 
```
There is no Claim for IRSA since it will be used as Nested Composition for Charts and Karpenter

### Deploy Charts
Configure vpcID in [Charts Claim](https://github.com/moonorb/crossplane-eks-karpenter/blob/main/examples/charts.yaml)
```
kubectl apply -f apis/charts/definition.yaml
kubectl apply -f apis/charts/composition.yaml
kubectl apply -f examples/charts.yaml
watch "crossplane beta trace charts  moonorb-test-x -n test-ns"

NAME                                                       SYNCED   READY   STATUS
Chart/moonorb-test-x (test-ns)                             True     True    Available
└─ XChart/moonorb-test-x-rvk2v                             True     True    Available
   ├─ XIRSA/moonorb-test-x-alb                             True     True    Available
   │  ├─ Policy/moonorb-test-x-rvk2v-4lqml                 True     True    Available
   │  ├─ RolePolicyAttachment/moonorb-test-x-rvk2v-kqq22   True     True    Available
   │  ├─ Role/moonorb-test-x-rvk2v-lxbq8                   True     True    Available
   │  └─ Object/moonorb-test-x-alb-controller-sa-role      True     True    Available
   ├─ XIRSA/moonorb-test-x-external-secrets                True     True    Available
   │  ├─ Policy/moonorb-test-x-rvk2v-nw9pz                 True     True    Available
   │  ├─ RolePolicyAttachment/moonorb-test-x-rvk2v-bvjjx   True     True    Available
   │  ├─ Role/moonorb-test-x-rvk2v-6jfsb                   True     True    Available
   │  └─ Object/moonorb-test-x-external-secrets-sa-role    True     True    Available
   ├─ Release/moonorb-test-x-alb                           True     True    Available
   └─ Release/moonorb-test-x-external-secrets              True     True    Available

```

### Deploy Karpenter
Configure Security Group and subnetID in [Karpenter Claim](https://github.com/moonorb/crossplane-eks-karpenter/blob/main/examples/eks.yaml) after EKS Cluster is provisioned
```
kubectl apply -f apis/karpenter/definition.yaml
kubectl apply -f apis/karpenter/composition.yaml
kubectl apply -f examples/karpenter.yaml
watch "crossplane beta trace karpenter moonorb-test-x -n test-ns"

NAME                                                       SYNCED   READY   STATUS
Karpenter/moonorb-test-x (test-ns)                         True     True    Available
└─ XKarpenter/moonorb-test-x-28z9j                         True     True    Available
   ├─ XIRSA/moonorb-test-x-karpenter                       True     True    Available
   │  ├─ Policy/moonorb-test-x-28z9j-hfdzx                 True     True    Available
   │  ├─ RolePolicyAttachment/moonorb-test-x-28z9j-dwrrp   True     True    Available
   │  ├─ Role/moonorb-test-x-28z9j-8npwb                   True     True    Available
   │  └─ Object/moonorb-test-x-karpenter-role              True     True    Available
   ├─ Rule/moonorb-test-x-healthevent                      True     True    Available
   ├─ Rule/moonorb-test-x-instancerebalance                True     True    Available
   ├─ Rule/moonorb-test-x-instancestatechange              True     True    Available
   ├─ Rule/moonorb-test-x-spotinterrupt                    True     True    Available
   ├─ Target/moonorb-test-x-28z9j-cdb2x                    True     True    Available
   ├─ Target/moonorb-test-x-28z9j-hfmsz                    True     True    Available
   ├─ Target/moonorb-test-x-28z9j-phqtz                    True     True    Available
   ├─ Target/moonorb-test-x-28z9j-sdcvm                    True     True    Available
   ├─ Release/moonorb-test-x-28z9j-9flgt                   True     True    Available
   ├─ InstanceProfile/moonorb-test-x-28z9j-cl25r           True     True    Available
   ├─ RolePolicyAttachment/moonorb-test-x-28z9j-9zc8f      True     True    Available
   ├─ RolePolicyAttachment/moonorb-test-x-28z9j-dhhc2      True     True    Available
   ├─ RolePolicyAttachment/moonorb-test-x-28z9j-h4cdm      True     True    Available
   ├─ RolePolicyAttachment/moonorb-test-x-28z9j-xf6sx      True     True    Available
   ├─ Role/moonorb-test-x-28z9j-csjjt                      True     True    Available
   ├─ Object/moonorb-test-x-28z9j-7gqs8                    True     True    Available
   ├─ Object/moonorb-test-x-28z9j-bcqjb                    True     True    Available
   ├─ Object/moonorb-test-x-28z9j-fb4jw                    True     True    Available
   ├─ QueuePolicy/moonorb-test-x-28z9j-gjjmc               True     True    Available
   └─ Queue/moonorb-test-x-28z9j-dfg4z                     True     True    Available

```

### Check Kubernetes events
At this point, you should not have any Warnings in your events
```
kubectl delete events --all=true
sleep 30
kubectl events --types=Warning
```

You can now connect to your newly created EKS Cluster after setting your ~/.kube/config: 
```
aws eks update-kubeconfig --name moonorb-test-x --region us-east-1  --no-verify-ssl
```

To switch back to the management cluster(kind):
```
kind export kubeconfig
```
