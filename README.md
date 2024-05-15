# Backstage Terraform Integrations

💥 Unlock the Power of Seamless Integration: Backstage Terraform Integrations Unleashed !

Prepare to embark on a transformative journey where innovation meets efficiency, and the boundaries of platform engineering are pushed to new heights. Welcome to the world of Backstage Terraform Integrations, where cnoe-io's reference implementations, such as idpbuilder and reference-implementations-aws, converge with the cutting-edge AWS-maintained, Terraform-based open-source solutions like data-on-eks and aws-observability-accelerator.

> **WORK IN PROGRESS**: *Current the repository is in POC stage, works only with [idpbuilder](https://github.com/cnoe-io/idpbuilder) and we only support creation of components from platform templates. We will continuously evolve this to add more more features such as supporting full lifecycle of components such as delete, update etc., and integrate newer AWS maintained terraform based open source solutions in future.*

## 🎯 About

![alt text](./images/backstage-integration-architecture.png)

Backstage Terraform integrations serve as a powerful bridge, seamlessly extending the capabilities of cnoe-io's reference implementations, such as [idpbuilder](https://github.com/cnoe-io/idpbuilder) and [reference-implementations-aws](https://github.com/cnoe-io/reference-implementation-aws). This integration enables these reference implementations to harness the full potential of AWS-maintained, Terraform-based open-source solutions, including [data-on-eks](https://github.com/awslabs/data-on-eks) and [aws-observability-accelerator](https://github.com/aws-observability/aws-observability-accelerator).

By leveraging these Terraform integrations, Backstage becomes a versatile platform that can effortlessly incorporate cutting-edge AWS technologies and services. This synergy empowers developers and architects to construct robust, scalable, and highly observable platform templates tailored to their specific needs.

The integration process is a symphony of automation and efficiency. Terraform, with its declarative approach to infrastructure as code, orchestrates the provisioning and configuration of the desired AWS resources. This streamlined process ensures consistency, reproducibility, and adherence to best practices across multiple environments, from development to production.

Moreover, the integration with AWS-maintained open-source solutions brings a wealth of expertise and community-driven innovation to the table. Solutions like data-on-eks and aws-observability-accelerator are meticulously crafted by AWS experts, incorporating industry-leading practices and architectural patterns. By harnessing these solutions, developers can benefit from battle-tested architectures, enhanced security, and optimized performance, all while reducing the time and effort required for implementation.

The seamless integration of Terraform and AWS-maintained open-source solutions within Backstage empowers organizations to unlock new realms of possibilities. Whether it's deploying highly available and scalable data platforms on Amazon Elastic Kubernetes Service (EKS) or implementing comprehensive observability solutions for monitoring, logging, and tracing, Backstage becomes a powerful catalyst for innovation and digital transformation.

With Backstage Terraform integrations, organizations can confidently embrace the latest AWS technologies, accelerate time-to-market, and deliver robust, secure, and highly observable platform templates that drive business value and competitive advantage.

## 🏃‍♀️ Prerequisites

1. We might need a container engines such as `Docker Desktop`, `Podman` to run backstage terraform integrations locally. Please check [this](https://github.com/cnoe-io/idpbuilder?tab=readme-ov-file#prerequisites) documentation to setup your container engine.

2. Download and install [idpbuilder](https://github.com/cnoe-io/idpbuilder?tab=readme-ov-file#download-and-install-the-idpbuilder) for running backstage terraform integrations.

## 🌟 Implementation walkthrough

1. Use the below command to deploy `idpbuilder` to make sure backstage terraform integration Argo application is deployed as part of your setup.

```bash
idpbuilder create \
  --use-path-routing \
  --package-dir examples/ref-implementation \
  --package-dir examples/terraform-integrations
```

2. Naviate to `idpbuilder` repo and create an AWS Secret on required namespaces for deploying templates on AWS environment using below commands:

```bash
export IDP_AWS_ACCESS_KEY_ID_BASE64=$(echo -n ${YOUR_AWS_ACCESS_KEY_ID} | base64)
export IDP_AWS_SECRET_ACCESS_KEY_BASE64=$(echo -n ${YOUR_AWS_SECRET_ACCESS_KEY} | base64)
export IDP_AWS_REGION=YOUR_AWS_REGION
# AWS Credentials for argo Namespace
cat << EOF > ./aws-secrets.yaml
---
apiVersion: v1
kind: Secret
metadata:
  name: aws-credentials
  namespace: argo
type: Opaque
data:
  AWS_ACCESS_KEY_ID: ${IDP_AWS_ACCESS_KEY_ID_BASE64}
  AWS_SECRET_ACCESS_KEY: $IDP_AWS_SECRET_ACCESS_KEY_BASE64
EOF
kubectl apply -f ./aws-secrets.yaml

# AWS Credentials for data-on-eks Namespace
cat << EOF > ./aws-secrets-doeks.yaml
---
apiVersion: v1
kind: Secret
metadata:
  name: aws-credentials
  namespace: data-on-eks
type: Opaque
data:
  AWS_ACCESS_KEY_ID: ${IDP_AWS_ACCESS_KEY_ID_BASE64}
  AWS_SECRET_ACCESS_KEY: $IDP_AWS_SECRET_ACCESS_KEY_BASE64
EOF
kubectl apply -f ./aws-secrets-doeks.yaml

# AWS Credentials for tf-eks-observability Namespace
cat << EOF > ./aws-secrets-eobs.yaml
---
apiVersion: v1
kind: Secret
metadata:
  name: aws-credentials
  namespace: tf-eks-observability
type: Opaque
data:
  AWS_ACCESS_KEY_ID: ${IDP_AWS_ACCESS_KEY_ID_BASE64}
  AWS_SECRET_ACCESS_KEY: $IDP_AWS_SECRET_ACCESS_KEY_BASE64
EOF
kubectl apply -f ./aws-secrets-eobs.yaml

# AWS Credentials for flux-system Namespace for TOFU Controller
cat << EOF > ./aws-secrets-tofu.yaml
---
apiVersion: v1
kind: Secret
metadata:
name: aws-credentials
namespace: flux-system
type: Opaque
stringData:
AWS_ACCESS_KEY_ID: ${YOUR_AWS_ACCESS_KEY_ID}
AWS_SECRET_ACCESS_KEY: ${YOUR_AWS_SECRET_ACCESS_KEY}
AWS_REGION: ${IDP_AWS_REGION}
EOF
kubectl apply -f ./aws-secrets-tofu.yaml
```

3. Next, in the `idpbuilder` folder, navigate to `./examples/ref-implementation/backstage/manifests/install.yaml` and add the following lines for catalog location at line 171 in backstage config to deploy terraform backstage templates to backstage:

```yaml
        - type: url
          target: https://github.com/cnoe-io/backstage-terraform-integrations/blob/main/backstage-templates-for-eks/catalog-info.yaml
          rules:
            - allow: [User, Group]
```

4. Finally, run the following `idpbuilder` command to build and run the terraform backstage integrations:

```bash
idpbuilder create \
  --use-path-routing \
  --package-dir examples/ref-implementation \
  --package-dir examples/terraform-integrations
```
## 🌟 Component delete workflow

Please follow the following steps if you are looking to delete a component created using the backstage terraform integrations :

1. In your [argocd](https://cnoe.localtest.me:8443/argocd) console, naviagate to your application created for your component and delete it manually.

2. In your [gitea](https://cnoe.localtest.me:8443/gitea/) console, the created repository for your component and delete it manually under settings.

3. Finally in your backstage console, navigate to component and click on `unregister component` to remove the deleted component from backstage.

## 🚀 Backstage and Argo UI

### Backstage environment with terraform templates

![Backstage Template Screen](./images/backstage-template.png)

### Argo App for Terraform Cluster Workflow Templates

![Argo Terraform Templates](./images/argo-terraform-template.png)

## 🤝 Contributing

If you'd like to contribute to the project or know the architecture and internals of this project, check out the [contribution doc](./CONTRIBUTING.md).

## 🔥 Extending the backstage terraform integrations

We will actively working to evolve this to add more more features such as supporting full lifecycle of components such as delete, update etc., and integrate newer AWS maintained terraform based open source solutions in future.

## 🙌 Community

We welcome all individuals who are enthusiastic about Kubernetes to become a part of this open source solution. Your contributions and participation are invaluable to the success of this project.

## 🙌 Collaboration

If you have any questions or need clarifications on topics covered here, please feel free to reach out to us on the [#cnoe-interest](https://cloud-native.slack.com/archives/C05TN9WFN5S) channel on CNCF Slack.

Built with ❤️.

## 🏆 Support & Feedback
Support is provided on a best effort basis. If you have feedback, feature ideas, or wish to report bugs, please use the [Issues](https://github.com/cnoe-io/backstage-terraform-integrations/issuess) section of this GitHub.