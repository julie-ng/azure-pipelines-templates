# Azure Pipelines - Snippets, Templates

Some re-usable [Azure Pipelines](https://azure.microsoft.com/en-us/services/devops/pipelines/) snippets I use acrossed my Azure Pipelines. 

For more about Azure DevOps Templates, please see official [Azure Docs on "template types & usage"](https://docs.microsoft.com/en-us/azure/devops/pipelines/process/templates?view=azure-devops). For an example pipeline, see my [azure-nodejs-demo](https://github.com/julie-ng/azure-nodejs-demo/blob/master/azure-pipelines.yml) project on GitHub.

## Setup

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

Because this a public repository, no service connection is required. For more information, see [Azure Docs: templates and using repositories](https://docs.microsoft.com/en-us/azure/devops/pipelines/process/templates?view=azure-devops#use-other-repositories)

## Templates

## `steps/set-custom-variable.yml`

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

_Note: `@templates` refers to repository name from setup as described above._

The result, for example `8cd076e` may be useful for logging and auditing your project by tagging images.

## `steps/docker-build-push.yml`

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

_Note: `@templates` refers to repository name from setup as described above._

#### Rant

Note the parameter is called `tagsAsMultilineString` which is due to limitations of Azure DevOps. A YAML object would make more sense to me. But [the underlying Azure DevOps task only takes multi-line strings](https://docs.microsoft.com/en-us/azure/devops/pipelines/tasks/build/docker?view=azure-devops#task-inputs) 🤷‍♀️


## `steps/lock-acr-image.yml`

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
