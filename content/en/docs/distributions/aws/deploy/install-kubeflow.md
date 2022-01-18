+++
title = "Install Kubeflow"
description = "Instructions for deploying Kubeflow on Amazon EKS"
weight = 20
+++

This guide describes how to use the `kubectl` and `kustomize` to deploy Kubeflow on Amazon Elastic Kubernetes Service (Amazon EKS) and Amazon Web Services (AWS).

Kubernetes versions 1.1x+ (JBDB) on Amazon EKS are compatible with Kubeflow version 1.3. Please see the [compatibility matrix](/docs/distributions/aws/deploy/eks-compatibility) for more information.

The installation below can be used with a new cluster, or you can install Kubeflow on an existing cluster. We recommend that you create a new cluster for better isolation. 

For information on customizing your deployment on AWS and Amazon EKS, please see [Customizing Kubeflow on AWS](/docs/distributions/aws/customizing-aws) for more information.

If you experience any issues with installation, see the [troubleshooting guidance](/docs/distributions/aws/troubleshooting-aws) for more information.

## Prerequisites

* Install [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl)
* Install and configure the AWS Command Line Interface (AWS CLI):
    * Install the [AWS Command Line Interface](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html).
    * Configure the AWS CLI by running the following command: `aws configure`.
    * Enter your Access Keys ([Access Key ID and Secret Access Key](https://docs.aws.amazon.com/general/latest/gr/aws-sec-cred-types.html#access-keys-and-secret-access-keys)).
    * Enter your preferred AWS Region and default output options.
* Install [eksctl](https://github.com/weaveworks/eksctl) and the [aws-iam-authenticator](https://docs.aws.amazon.com/eks/latest/userguide/install-aws-iam-authenticator.html).

## EKS cluster

Before moving forward with Kubeflow installation on AWS, you will need a Kubernetes cluster. The simplest way to get started is to create a cluster in Amazon EKS. If you already have a cluster, ensure that your current `kubectl` context is set and move on to the next step.

There are several ways to provision a cluster in EKS, including with the `aws` CLI, in the EKS Console, or via AWS CloudFormation, Terraform, or the AWS Cloud Development Kit (CDK). A simple way to get started is by using [eksctl](https://github.com/weaveworks/eksctl), which we recommend here.

First, set a few environment variables to specify your desired cluster name, AWS region, Kubernetes version, and Amazon EC2 instance type to use for cluster nodes.

For example:

```shell
export AWS_CLUSTER_NAME=kubeflow-demo
export AWS_REGION=us-west-2
export K8S_VERSION=1.18
export EC2_INSTANCE_TYPE=m5.large
```

Now, create a cluster configuration file for use with `eksctl`.

```shell
cat << EOF > cluster.yaml
---
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: ${AWS_CLUSTER_NAME}
  version: "${K8S_VERSION}"
  region: ${AWS_REGION}

managedNodeGroups:
- name: kubeflow-mng
  desiredCapacity: 3
  instanceType: ${EC2_INSTANCE_TYPE}
EOF
```

Finally, create the cluster using `eksctl`.

```shell
eksctl create cluster -f cluster.yaml
```

## Prerequisites

* Ensure you have a default [StorageClass](https://kubernetes.io/docs/concepts/storage/storage-classes/) in your Kubernetes cluster. This will be in place already with an EKS cluster.
* [kubectl](https://kubernetes.io/docs/tasks/tools/#kubectl) installed and configured for your cluster
* kustomize [version 3.2.0](https://github.com/kubernetes-sigs/kustomize/releases/tag/v3.2.0) installed

## Deploy Kubeflow

In recent versions, Kubeflow distribution manifests are maintained outside of the Kubeflow project. To prepare for deployment on AWS, clone the [AWS Kubeflow Manifests](https://github.com/awslabs/kubeflow-manifests) project locally and checkout the version you wish to install.

```shell
git clone https://github.com/awslabs/kubeflow-manifests kubeflow-manifests
cd kubeflow-manifests
git checkout v1.3-branch
```

You can install all Kubeflow official components (residing under `apps`) and all common services (residing under `common`) using the following command:

```shell
while ! kustomize build example | kubectl apply -f - | grep -v unchanged; do echo "Retrying to apply resources"; sleep 10; done
```

## Deployment options

Deployment of Kubeflow on AWS is customizable, and depending upon your use case you may choose to integrate your deployment with AWS services.

### Authentication

The default deployment will leverage [Dex](https://dexidp.io/), an OpenID Connect provider. See the [AWS Kubeflow Manifests](https://github.com/awslabs/kubeflow-manifests#dex) component notes for more information.

Optionally, you may deploy Kubeflow with an integration for [AWS Cognito](https://aws.amazon.com/cognito/) for your authentication needs. Refer to the [Deploying Kubeflow with AWS Cognito as idP](https://github.com/awslabs/kubeflow-manifests/tree/v1.3-branch/distributions/aws/examples/cognito) guide.

### Component integrations

Kubeflow components on AWS can be deployed with integrations for [Amazon S3](https://aws.amazon.com/s3/) and [Amazon RDS](https://aws.amazon.com/rds/). Refer to the [Kustomize Manifests for RDS and S3](https://github.com/awslabs/kubeflow-manifests/tree/v1.3-branch/distributions/aws/examples/rds-s3) guide for deployment configuration instructions.

For convenience, there is also a single guide for deploying Kubeflow on AWS with [RDS, S3, and Cognito](https://github.com/awslabs/kubeflow-manifests/tree/v1.3-branch/distributions/aws/examples/cognito-rds-s3).

### Persistent storage

Along with Kubernetes support for Amazon EBS, Kubeflow on AWS has integrations for using [Amazon EFS](https://aws.amazon.com/efs/) or [Amazon FSx for Lustre](https://aws.amazon.com/fsx/lustre/) for persistent storage.

Amazon EFS supports `ReadWriteMany` access mode, which means the volume can be mounted as read-write by many nodes. This is useful for creating a shared filesystem that can be mounted into multiple pods, as you may have with Jupyter. For example, one group can share datasets or models across an entire team.

Amazon FSx for Lustre provides a high-performance file system optimized for fast processing for machine learning and high performance computing (HPC) workloads.  Lustre also supports `ReadWriteMany`. One difference between Amazon EFS and Lustre is that Lustre can be used to cache training data with direct connectivity to Amazon S3 as the backing store. With this configuration, you don't need to transfer data to the file system before using the volume.

Refer to the respective guides for [Amazon EFS](https://github.com/awslabs/kubeflow-manifests/tree/v1.3-branch/distributions/aws/examples/storage/efs) and [Amazon FSx for Lustre](https://github.com/awslabs/kubeflow-manifests/tree/v1.3-branch/distributions/aws/examples/storage/fsx-for-lustre) for more details.

For additional configuration and deployment options, see the [AWS Kubeflow Manifests documentation](https://github.com/awslabs/kubeflow-manifests/#installation).

## Working with your Kubeflow deployment

It will take some time for the deployment to complete. Make sure all Pods are ready before trying to connect, otherwise you might get unexpected errors. To check that all Kubeflow-related Pods are ready, use the following commands:

```shell
kubectl get pods -n cert-manager
kubectl get pods -n istio-system
kubectl get pods -n auth
kubectl get pods -n knative-eventing
kubectl get pods -n knative-serving
kubectl get pods -n kubeflow
kubectl get pods -n kubeflow-user-example-com
```

For instructions on connecting to your Kubeflow deployment, see the [AWS Kubeflow manifest documentation](https://github.com/awslabs/kubeflow-manifests/#connect-to-your-kubeflow-cluster).

Ensure that you [change the default password](https://github.com/awslabs/kubeflow-manifests/#change-default-user-password) as shown as well.

## Post Installation

Kubeflow provides multi-tenancy support and users are not able to create notebooks in either the `kubeflow` or `default` namespaces.

During the first user login, the system will create the `anonymous` namespace. To create additional users, you can create profiles.

Create a manifest file `profile.yaml` with the following contents.

```yaml
apiVersion: kubeflow.org/v1beta1
kind: Profile
metadata:
  name: test
spec:
  owner:
    kind: User
    name: test@amazon.com
```

> Note: `spec.owner.name` is the user's email.

Now create the profile via `kubectl create -f profile.yaml`. The Profile controller will create a new namespace `name` and related service account to allow for notebook creation in that namespace.

Check [Multi-Tenancy in Kubeflow](/docs/components/multi-tenancy) for more details.