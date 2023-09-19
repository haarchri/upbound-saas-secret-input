# upbound-saas-secret-input

We require this procedure today in Upbound SaaS because environment configurations are not activated by default. Currently, these environment configurations are hidden behind an alpha flag. It's a simple adjustment to make in the future once environment configurations become stable and available in upbound-saas.

## Overview

This configuration showcases the ability to utilize static input fields from Kubernetes secret. 
To achieve this, we require a combination of the following components:
    - provider-upbound
    - provider-kubernetes
    - and customer-chosen providers. 

The initial step involves self importing the MCP controlplane to obtain a valid kubeconfig. Without this, provider-kubernetes won't be able to access resources from the MCP controlplane. Once the controlplane is imported, we can input a secret and populate all the fields in the status fields of a composite. These status fields can then be used for patches in the composition process.

The concept here is that the secret, serving as input for each cluster, is managed and written by GitOps tools such as Flux or ArgoCD to the MCP control plane. This enables you to synchronize it from your internal Git repositories.

## PreReqs:

To begin, please create a provider configuration and a secret for provider-upbound. This setup will allow you to independently import your MCP control plane.
https://github.com/upbound/provider-upbound/tree/main/examples/provider

## Steps:

```bash
kubectl apply -f .up/examples/controlplane.yaml
kubectl apply -f .up/examples/secret.yaml
kubectl apply -f .up/examples/example.yaml
```

## Outputs:

```bash
kubectl get xinput input-test-8v4bg -o yaml
apiVersion: haarchri.io/v1
kind: XInput
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"haarchri.io/v1","kind":"Input","metadata":{"annotations":{},"name":"input-test","namespace":"default"},"spec":{"clusterId":"dev-cluster-a","eksVersion":"1.27","mcpName":"haarchri-display-nothing","writeConnectionSecretToRef":{"name":"input-test-conn"}}}
  creationTimestamp: "2023-09-19T07:30:42Z"
  finalizers:
  - composite.apiextensions.crossplane.io
  generateName: input-test-
  generation: 11
  labels:
    crossplane.io/claim-name: input-test
    crossplane.io/claim-namespace: default
    crossplane.io/composite: input-test-8v4bg
  name: input-test-8v4bg
  resourceVersion: "495903"
  uid: c562a57a-848c-4319-a5a6-6ef210f9fd2f
spec:
  claimRef:
    apiVersion: haarchri.io/v1
    kind: Input
    name: input-test
    namespace: default
  clusterId: dev-cluster-a
  compositionRef:
    name: xinput.haarchri.io
  compositionRevisionRef:
    name: xinput.haarchri.io-c2ee327
  compositionUpdatePolicy: Automatic
  eksVersion: "1.27"
  mcpName: haarchri-display-nothing
  resourceRefs:
  - apiVersion: kubernetes.crossplane.io/v1alpha1
    kind: ProviderConfig
    name: dev-cluster-a
  - apiVersion: kubernetes.crossplane.io/v1alpha1
    kind: Object
    name: input-test-8v4bg-9t24h
  - apiVersion: nop.crossplane.io/v1alpha1
    kind: NopResource
    name: input-test-8v4bg-792b8
  writeConnectionSecretToRef:
    name: c562a57a-848c-4319-a5a6-6ef210f9fd2f
    namespace: upbound-system
status:
  conditions:
  - lastTransitionTime: "2023-09-19T07:30:43Z"
    reason: ReconcileSuccess
    status: "True"
    type: Synced
  - lastTransitionTime: "2023-09-19T07:46:47Z"
    reason: Available
    status: "True"
    type: Ready
  connectionDetails:
    lastPublishedTime: "2023-09-19T07:30:43Z"
  status:
    adminRoleArn: arn:aws:iam::123456789101:role/AWSReservedSSO_AdministratorAccess_123456789defg
    subnetA: subnet-05e12583aa5eb7f1a
    subnetB: subnet-05e12583aa5eb7f1b
    subnetC: subnet-05e12583aa5eb7f1c
    vpcId: vpc-12345678910
```

```bash
kubectl get nopresource.nop.crossplane.io/input-test-8v4bg-792b8  -o yaml
apiVersion: nop.crossplane.io/v1alpha1
kind: NopResource
metadata:
  annotations:
    crossplane.io/composition-resource-name: NopResource
    crossplane.io/external-name: input-test-8v4bg-792b8
  creationTimestamp: "2023-09-19T07:30:43Z"
  finalizers:
  - finalizer.managedresource.crossplane.io
  generateName: input-test-8v4bg-
  generation: 3
  labels:
    crossplane.io/claim-name: input-test
    crossplane.io/claim-namespace: default
    crossplane.io/composite: input-test-8v4bg
  name: input-test-8v4bg-792b8
  ownerReferences:
  - apiVersion: haarchri.io/v1
    blockOwnerDeletion: true
    controller: true
    kind: XInput
    name: input-test-8v4bg
    uid: c562a57a-848c-4319-a5a6-6ef210f9fd2f
  resourceVersion: "495906"
  uid: 22e7e45c-bd45-4a9d-b4a8-c2fcddac577c
spec:
  deletionPolicy: Delete
  forProvider:
    fields:
      combineField: |
        dev-cluster-a, 1.27, vpc-12345678910, subnet-05e12583aa5eb7f1a, subnet-05e12583aa5eb7f1b, subnet-05e12583aa5eb7f1c, arn:aws:iam::123456789101:role/AWSReservedSSO_AdministratorAccess_123456789defg
      stringField: use secret input
  providerConfigRef:
    name: default
  writeConnectionSecretToRef:
    name: c562a57a-848c-4319-a5a6-6ef210f9fd2f-nop
    namespace: default
status:
  atProvider: {}
  conditions:
  - lastTransitionTime: "2023-09-19T07:30:43Z"
    reason: ReconcileSuccess
    status: "True"
    type: Synced
```