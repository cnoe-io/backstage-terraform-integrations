# Backstage Terraform Integrations

1. [idpbuilder](https://github.com/cnoe-io/idpbuilder/tree/main/examples/ref-implementation) setup is a pre-requisite to run this terraform backstate integration solution. Naviate to `idpbuilder` repo and create an AWS Secret for deploying templates on AWS environment using below commands:

```bash
export IDP_AWS_ACCESS_KEY_ID_BASE64=$(echo -n ${YOUR_AWS_ACCESS_KEY_ID} | base64)
export IDP_AWS_SECRET_ACCESS_KEY_BASE64=$(echo -n ${YOUR_AWS_SECRET_ACCESS_KEY} | base64
cat << EOF > ./examples/ref-implementation/argo-workflows/manifests/dev/aws-secrets.yaml
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
kubectl apply -f ./examples/ref-implementation/argo-workflows/manifests/dev/aws-secrets.yaml
```

2. Next, lets create a GitHub App Integration with `idpbuilder` setup to create GitHUb repos as part of template deployments. First lets create a GitHub Application to build an integration secret. GitHub app is used to enable integration between Backstage and GitHub.
This allows you for integration actions such as automatically importing Backstage configuration such as Organization information and templates.

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

3. Next, In the `idpbuilder` folder, navigate to `./examples/ref-implementation/backstage/manifests/install.yaml` and uncomment the following lines to enable the created GitHub Integration in previous step in backstage config.

```yaml
      github:
        - host: github.com
          apps:
            - $include: github-integration.yaml
```

4. Next, In the same file, add the following lines after line 280 for GitHub Integration Secret:

```yaml
              - secret:
                  name: integrations
                  items:
                    - key: github-integration.yaml
                      path: github-integration.yaml
```

5. Next, run the below kubectl command to create an Argo application for create backstage `ClusterWorkFlow` Templates:

```bash
https://raw.githubusercontent.com/cnoe-io/backstage-terraform-integrations/main/argo-application/terraform-workflows-templates.yaml
```

6. Next, In the `idpbuilder` folder, navigate to `./examples/ref-implementation/backstage/manifests/install.yaml` and add the following lines for catalog location at line 171 in backstage config to deploy terraform backstage templates to backstage:

```yaml
        - type: url
          target: https://github.com/cnoe-io/backstage-terraform-integrations/blob/main/backstage-templates-for-eks/catalog-info.yaml
          rules:
            - allow: [User, Group]
```

7. Finally, run the `idpbuilder` command `idpbuilder create --use-path-routing --package-dir examples/ref-implementation -n` to incrementally build and run the terraform backstage integrations.

## Screenshots

### Backstage environment with terraform templates

![Backstage Template Screen](./images/backstage-template.png)

### Argo Terraform Templates

![Argo Terraform Templates](./images/argo-terraform-template.png)
