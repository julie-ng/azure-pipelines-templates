# Re-usable Templates for Azure Pipelines

Some re-usable [Azure Pipelines](https://azure.microsoft.com/en-us/services/devops/pipelines/) snippets I use acrossed my Azure Pipelines. 

| Name | Template | 
|:--|:--|
| [Set Custom Variable](#set-custom-variable) | `steps/set-custom-variable.yml`|
| [Combined Docker login, build, push and logout Tasks](#combined-docker-login-build-push-and-logout-tasks) | `steps/docker-build-push.yml`| 
| [Lock Azure Container Registory Image](#lock-azure-container-registory-image) (makes immutable) | `steps/lock-acr-image.yml`| 
| [Deploy Container to Azure App Service](#deploy-container-to-azure-app-service) | `steps/deploy-app-service.yml`| 

### References 

- Official Docs - [Azure DevOps > Template types & usage"](https://docs.microsoft.com/en-us/azure/devops/pipelines/process/templates?view=azure-devops)
- Example pipeline - [julie-ng/azure-nodejs-demo/azure-pipelines.yml](https://github.com/julie-ng/azure-nodejs-demo/blob/master/azure-pipelines.yml)


## Setup - Include Library

In your `azure-pipelines.yml` file for your project, add a reference to this repository:

```yaml
# File: azure-pipelines.yml

resources:
  repositories:
    - repository: templates # for reference
      type: github
      name: julie-ng/azure-pipelines-templates
      ref: refs/heads/master      
```

_Note: `@templates` suffix always refers to repository name from setup as described above._

### Set Custom Variable

Sometimes it's helpful to set a custom variable `at runtime` based on output. For example, you need project version or the git commit of the build. This task encapsulates the clunky `##vso[task.setvariable…]` syntax for you.

For example, here we set the `git-sha` variable to output of `git rev-parse --short HEAD` command:

```yaml
# File: azure-pipelines.yml

steps:
- template: steps/set-custom-variable.yml@templates
  parameters:
    variableName: git-sha
    command: 'git rev-parse --short HEAD'				
```

The result, for example `8cd076e` may be useful for logging and auditing your project by tagging images.

### Combined Docker login, build, push and logout Tasks

This template combines the following docker tasks:

- registry login
- build and tag image
- push image with tag(s)
- registry logout

The example below takes the `$(git-sha)` variable and uses it as a Docker tag. This example pushes a single image with tagged both by its git sha and the version `1.0.0`

```yaml
steps:
- template: steps/docker-build-push.yml@templates
  parameters:
    registryConnectionName: 'acr-or-docker-hub-connection'
    imageName: 'my-app' 
    tagsAsMultilineString: |
      $(git-sha)
      '1.0.0'
```

Note the parameter is called `tagsAsMultilineString` which is due to limitations of Azure DevOps. A YAML object would make more sense to me. But [the underlying Azure DevOps task only takes multi-line strings](https://docs.microsoft.com/en-us/azure/devops/pipelines/tasks/build/docker?view=azure-devops#task-inputs) 🤷‍♀️

### Lock Azure Container Registory Image

This template [locks an image in the Azure Container Registry](https://docs.microsoft.com/en-us/azure/container-registry/container-registry-image-lock) preventing it from being deleted.

Example use:

```yaml
steps:
- template: steps/lock-acr-image.yml@templates
  parameters:
    azureArmConnection: 'azure-arm-connection'
    acrRegistryName: 'myregistry'
    imageName: 'my-app'
    imageTag: 'v1.0'
    condition: succeeded() # optional
```

### Deploy Container to Azure App Service

Deploys a container to Azure App Service, [Web App for Containers](https://azure.microsoft.com/en-us/services/app-service/containers/). Works for both Docker Hub and Azure Container Registry images.

Example use:

```yaml
steps:
  - template: steps/deploy-app-service.yml@templates
    parameters:
      ARMConnectionName: 'arm-service-connection'
      dockerImage: 'image-name:v1.0'
      appName: 'myapp-dev'
```