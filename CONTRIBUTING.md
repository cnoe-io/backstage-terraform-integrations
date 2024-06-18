# Contributing guide

Welcome to the project. Thank you for your interest in contributing to our project. Whether it's a bug report, new feature, correction, or additional documentation, we greatly value feedback and contributions from our community.

Please read through this document before submitting any issues or pull requests to ensure we have all the necessary
information to effectively respond to your bug report or contribution.

If you have any questions or need clarifications on topics covered here, please feel free to reach out to us on the [#cnoe-interest](https://cloud-native.slack.com/archives/C05TN9WFN5S) channel on CNCF Slack.

Please check on our Contributing guide in [idpbuilder](https://github.com/cnoe-io/idpbuilder/blob/main/CONTRIBUTING.md) to learn about contributing to `idpbuilder`.

## Project Information

### Integrating an AWS maintained terraform open source solutions to backstage terraform integrations:

1. Make sure the existing AWS maintained terraform open source solutions is popular and helps the community and customers. Please use this guideline on [Templating of Terraform Modules](https://cnoe.io/docs/reference-implementation/extensions/tf-templating) to templatize existingAWS maintained terraform open source solution.
2. Create a new `ClusterWorkflowTemplate` and other required supporting Kubernetes resource such as namespace, service account etc in [base](./argo-workflows-templates/base) folder. Make sure to add your new files to [kustomization file](./argo-workflows-templates/base/kustomization.yaml).
3. Create a new folder under `backstage-templates-for-eks` if your integration is a non-existent AWS maintained terraform open source solution and create sub folder for the respective solution and create required files such as `catalog-info.yaml` and manifests to support your solution. You can take a look in to our `data-on-eks` and `eks-observability-accelerator` examples.
4. Finally update the [catalog-info.yaml](./backstage-templates-for-eks/catalog-info.yaml) with references to your new templates of your solution integration.

## Reporting Bugs/Feature Requests

We welcome you to use the GitHub issue tracker to report bugs or suggest features.

When filing an issue, please check existing open, or recently closed, issues to make sure somebody else hasn't already
reported the issue. Please try to include as much information as you can. Details like these are incredibly useful:

* A reproducible test case or series of steps
* The version of our code being used
* Any modifications you've made relevant to the bug
* Anything unusual about your environment or deployment

## Contributing via Pull Requests
Contributions via pull requests are much appreciated. Before sending us a pull request, please ensure that:

1. You are working against the latest source on the *main* branch.
2. You check existing open, and recently merged, pull requests to make sure someone else hasn't addressed the problem already.
3. You open an issue to discuss any significant work - we would hate for your time to be wasted.

To send us a pull request, please:

1. Fork the repository.
2. Modify the source; please focus on the specific change you are contributing. If you also reformat all the code, it will be hard for us to focus on your change.
3. Ensure local tests pass.
4. Commit to your fork using clear commit messages.
5. Send us a pull request, answering any default questions in the pull request interface.
6. Pay attention to any automated CI failures reported in the pull request, and stay involved in the conversation.

GitHub provides additional document on [forking a repository](https://help.github.com/articles/fork-a-repo/) and
[creating a pull request](https://help.github.com/articles/creating-a-pull-request/).



