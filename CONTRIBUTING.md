# Contributing guide

Welcome to the project, and thanks for considering contributing to this project. 

If you have any questions or need clarifications on topics covered here, please feel free to reach out to us on the [#cnoe-interest](https://cloud-native.slack.com/archives/C05TN9WFN5S) channel on CNCF Slack.

Please check on our Contributing guide in [idpbuilder](https://github.com/cnoe-io/idpbuilder/blob/main/CONTRIBUTING.md) to learn about contributing to `idpbuilder`.

## Project Information

### Integrating an AWS maintained terraform open source solutions to backsstage terraform integrations:

1. Make sure the AWS maintained terraform open source solutions is popular and helps the community and customers.
2. Create a new `ClusterWorkflowTemplate` and other required supporting Kubernetes resource such as namespace, service account etc in [base](./argo-workflows-templates/base) folder. Make sure to add your new files to [kustomization file](./argo-workflows-templates/base/kustomization.yaml).
3. Create a new folder under `backstage-templates-for-eks` if your integration is a non-existant AWS maintained terraform open source solution and create sub folder for the respective solution and create required files such as `catalog-info.yaml` and manifests to support your solution. You can take a look in to our `data-on-eks` and `eks-observability-accelerator` examples.
4. Finally update the [catalog-info.yaml](./backstage-templates-for-eks/catalog-info.yaml) with references to your new templates of your solution integration.

