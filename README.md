# Backstage Terraform Integrations

Backstage terraform integrations helps extends cnoe-io reference implementations to support newer platform templates on backstage.extends cnoe-io reference implementations such as [idpbuilder](https://github.com/cnoe-io/idpbuilder), [reference-implementations-aws](https://github.com/cnoe-io/reference-implementation-aws) to integrate with AWS maintained terraform based open source solutions such as [data-on-eks](https://github.com/awslabs/data-on-eks), [aws-observability-accelerator](https://github.com/aws-observability/aws-observability-accelerator) to support newer platform templates on backstage.

> **WORK IN PROGRESS**: Current the repository is in POC stage and we only support creation of components from platform templates. We will continuously evolve this to add more more features such as supporting full lifecycle of components such as delete, update etc., and integrate newer AWS maintained terraform based open source solutions in future.

## About

Backstage Terraform integrations serve as a powerful bridge, seamlessly extending the capabilities of cnoe-io's reference implementations, such as [idpbuilder](https://github.com/cnoe-io/idpbuilder) and [reference-implementations-aws](https://github.com/cnoe-io/reference-implementation-aws). This integration enables these reference implementations to harness the full potential of AWS-maintained, Terraform-based open-source solutions, including [data-on-eks](https://github.com/awslabs/data-on-eks) and [aws-observability-accelerator](https://github.com/aws-observability/aws-observability-accelerator).

By leveraging these Terraform integrations, Backstage becomes a versatile platform that can effortlessly incorporate cutting-edge AWS technologies and services. This synergy empowers developers and architects to construct robust, scalable, and highly observable platform templates tailored to their specific needs.

The integration process is a symphony of automation and efficiency. Terraform, with its declarative approach to infrastructure as code, orchestrates the provisioning and configuration of the desired AWS resources. This streamlined process ensures consistency, reproducibility, and adherence to best practices across multiple environments, from development to production.

Moreover, the integration with AWS-maintained open-source solutions brings a wealth of expertise and community-driven innovation to the table. Solutions like data-on-eks and aws-observability-accelerator are meticulously crafted by AWS experts, incorporating industry-leading practices and architectural patterns. By harnessing these solutions, developers can benefit from battle-tested architectures, enhanced security, and optimized performance, all while reducing the time and effort required for implementation.

The seamless integration of Terraform and AWS-maintained open-source solutions within Backstage empowers organizations to unlock new realms of possibilities. Whether it's deploying highly available and scalable data platforms on Amazon Elastic Kubernetes Service (EKS) or implementing comprehensive observability solutions for monitoring, logging, and tracing, Backstage becomes a powerful catalyst for innovation and digital transformation.

With Backstage Terraform integrations, organizations can confidently embrace the latest AWS technologies, accelerate time-to-market, and deliver robust, secure, and highly observable platform templates that drive business value and competitive advantage.

## Prerequisites

1. We might need a container engines such as `Docker Desktop`, `Podman` to run backstage terraform integrations locally. Please check [this](https://github.com/cnoe-io/idpbuilder?tab=readme-ov-file#prerequisites) documentation to setup your container engine.

2. Download and install [idpbuilder](https://github.com/cnoe-io/idpbuilder?tab=readme-ov-file#download-and-install-the-idpbuilder) for running backstage terraform integrations.

## Implementation walkthrough

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
```

3. Next, lets create a GitHub App Integration with `idpbuilder` setup to create GitHUb repos as part of template deployments. First lets create a GitHub Application to build an integration secret. GitHub app is used to enable integration between Backstage and GitHub. This allows you for integration actions such as automatically importing Backstage configuration such as Organization information and templates.

We strongly encourage you to create a **dedicated GitHub organization**. If you don't have an organization for this purpose, please follow [this link](https://docs.github.com/en/organizations/collaborating-with-groups-in-organizations/creating-a-new-organization-from-scratch) to create one.

There are two ways to create GitHub integration with Backstage. You can use the Backstage CLI, or create it manually. See [this page](https://backstage.io/docs/integrations/github/github-apps) for more information on creating one manually. Once the app is created, place it under the private directory with the name `github-integration.yaml`. 

To create one with the CLI, follow the steps below. If you are using cli to create GitHub App, please make sure to select third option in the permissions prompt, if your GitHub App access needs publishing access to create GitHub repositories for your backstage templates.

```bash
npx '@backstage/cli' create-github-app ${GITHUB_ORG_NAME}
# If prompted, select all for permissions or select permissions listed in this page https://backstage.io/docs/integrations/github/github-apps#app-permissions
# In the browser window, allow access to all repositories then install the app.

? Select permissions [required] (these can be changed later but then require approvals in all installations) (Press <space> to select, <a> to toggle all, <i> to invert selection,
and <enter> to proceed)
 ◉ Read access to content (required by Software Catalog to ingest data from repositories)
 ◉ Read access to members (required by Software Catalog to ingest GitHub teams)
❯◯ Read and Write to content and actions (required by Software Templates to create new repositories)

# move it to a "private" location. 
mkdir -p private
GITHUB_APP_FILE=$(ls github-app-* | head -n1)
mv ${GITHUB_APP_FILE} private/github-integration.yaml
export GITHUB_APP_YAML_INDENTED=$(cat ./private/github-integration.yaml | base64 | sed 's/^/    /')

cat << EOF > ./github-integrations-secret.yaml
---
apiVersion: v1
kind: Secret
metadata:
  name: integrations
  namespace: backstage
data:
  github-integration.yaml: |
${GITHUB_APP_YAML_INDENTED}
EOF
kubectl apply -f ./github-integrations-secret.yaml
```

4. Next, in the `idpbuilder` folder, navigate to `./examples/ref-implementation/backstage/manifests/install.yaml` and uncomment the following lines to enable the created GitHub Integration in previous step in backstage config.

```yaml
      github:
        - host: github.com
          apps:
            - $include: github-integration.yaml
```

5. Next, in the same file, add the following lines after line 280 for GitHub Integration Secret:

```yaml
              - secret:
                  name: integrations
                  items:
                    - key: github-integration.yaml
                      path: github-integration.yaml
```

6. Next, in the `idpbuilder` folder, navigate to `./examples/ref-implementation/backstage/manifests/install.yaml` and add the following lines for catalog location at line 171 in backstage config to deploy terraform backstage templates to backstage:

```yaml
        - type: url
          target: https://github.com/cnoe-io/backstage-terraform-integrations/blob/main/backstage-templates-for-eks/catalog-info.yaml
          rules:
            - allow: [User, Group]
```

7. Finally, run the following `idpbuilder` command to build and run the terraform backstage integrations:

```bash
idpbuilder create \
  --use-path-routing \
  --package-dir examples/ref-implementation \
  --package-dir examples/terraform-integrations
```

## Screenshots

### Backstage environment with terraform templates

![Backstage Template Screen](./images/backstage-template.png)

### Argo App for Terraform Cluster Workflow Templates

![Argo Terraform Templates](./images/argo-terraform-template.png)
