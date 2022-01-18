+++
title = "Amazon EKS and Kubeflow Compatibility"
description = "Compatibility matrix for Kubeflow on Amazon EKS by Kubernetes version"
weight = 25
+++

## Compatibility

Starting with Kubeflow version 1.2, Amazon EKS maintains end-to-end testing between EKS Kubernetes versions and Kubeflow versions. The following table relates compatibility between Kubernetes versions on Amazon EKS and Kubeflow version v1.3 (XXX shortcode).

<div class="table-responsive">
  <table class="table table-bordered">
    <thead class="thead-light">
      <tr>
        <th>EKS Versions</th>
        <th>Kubeflow {{% aws/kfctl-aws %}} XXX need a new shortcode for version</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td>1.19</td>
        <td><b>Compatible</b></td>
      </tr>
      <tr>
        <td>1.20</td>
        <td><b>Compatible</b></td>
      </tr>
      <tr>
        <td>1.21</td>
        <td><b>Compatible</b></td>
      </tr>
    </tbody>
  </table>
</div>

- **Incompatible**: the combination is not known to work together
- **Compatible**: all Kubeflow features have been tested and verified for the EKS Kubernetes version
- **No known issues**: the combination has not been fully tested but there are no reported issues
