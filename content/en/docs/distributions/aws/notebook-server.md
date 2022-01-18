+++
title = "Optimized Jupyter Notebooks on AWS"
description = "AWS-optimized Notebooks based on AWS Deep Learning Containers"
weight = 40
+++

## AWS Optimized Notebook Images

Installing Kubeflow on AWS using this guide will include AWS-optimized Kubeflow Notebook Images as the default options in the notebook server.

These images are based upon [AWS Deep Learning Containers](https://docs.aws.amazon.com/deep-learning-containers/latest/devguide/what-is-dlc.html). AWS Deep Learning Containers provide optimized environments with TensorFlow and MXNet, Nvidia CUDA (for GPU instances), and Intel MKL (for CPU instances) libraries.

Additional pre-installed packages:
- `docker-client`
- `kubeflow-metadata`
- `kfp`
- `kfserving`

Refer to the [documentation](https://docs.aws.amazon.com/deep-learning-containers/latest/devguide/deep-learning-containers-images.html) and the upstream [project on GitHub](https://github.com/aws/deep-learning-containers) for the list of [available container images](https://github.com/aws/deep-learning-containers/blob/master/available_images.md).