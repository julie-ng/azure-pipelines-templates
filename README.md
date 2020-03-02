# azure-pipeline-templates

Some re-usable Azure Pipelines snippets I use.


## Setup

In your `azure-pipelines.yml` file for your project, add a reference to this repository:

```yaml
# File: azure-pipelines.yml

resources:
  repositories:
    - repository: templates
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
		registryConnectionName: $(registry-service-connection)
		imageName: $(image-name)
		tagsAsMultilineString: |
			$(git-sha)
			'1.0.0'
```

_Note: `@templates` refers to repository name from setup as described above._

#### Rant

Note the parameter is called `tagsAsMultilineString` which is due to limitations of Azure DevOps. A YAML object would make more sense to me. But [the underlying Azure DevOps task only takes multi-line strings](https://docs.microsoft.com/en-us/azure/devops/pipelines/tasks/build/docker?view=azure-devops#task-inputs) 🤷‍♀️
