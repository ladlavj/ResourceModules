This section provides an overview of the design principles applied to the CARML CI environment pipelines.

---

### _Navigation_

- [Module Pipelines](#module-pipelines)
  - [Pipeline phases](#pipeline-phases)
    - [DevOps-Tool-specific design](#devops-tool-specific-design)
  - [Module pipeline inputs](#module-pipeline-inputs)
- [Platform pipelines](#platform-pipelines)
  - [ReadMe pipeline](#readme-pipeline)
  - [Wiki pipeline](#wiki-pipeline)

---

# Module Pipelines

The repository hosts one pipeline for each module in the CARML library.

The purpose of each module pipeline is twofold:

1. **Validation**: To ensure the modules hosted by the CARML library are valid and can perform the intended deployments.
1. **Publishing**: To publish _versioned_ and already validated modules to one or multiple target locations, from where they can be referenced by solutions consuming them.

As such, each pipeline can be mapped to `Phases 1 and 2` described in the [Deployment flow](./The%20context%20-%20CARML%20CI%20environment#deployment-flow) section.

<img src="./media/CIEnvironment/pipelineDesign.png" alt="Pipeline phases" height="500">

The following paragraphs provide an overview of the different phases and shared logic the module pipelines use.

- [Pipeline phases](#pipeline-phases)
- [Module pipeline inputs](#module-pipeline-inputs)

## Pipeline phases

This paragraph provides an overview of the three phases performed by each module pipeline. Further details about the implementation and design of each phase are provided on the dedicated pages linked below.

1. **Static Validation**: Runs a set of static Pester tests on the module and its templates to ensure they comply with the design principles of CARML. Further details for this phase are provided on the corresponding wiki page - see the [Static validation](./The%20CI%20environment%20-%20Static%20validation) section.
1. **Deployment Validation**: An actual Azure deployment is run in a sandbox subscription leveraging a predefined set of module test files, each validating a different configuration of the same Azure resource in parallel. The test suite is cleaned up by default, removing all test resources post-deployment. Further details for this phase are provided on the corresponding wiki page - see the [Deployment validation](./The%20CI%20environment%20-%20Deployment%20validation) section.
1. **Publishing**: Runs only if the previous steps are successful. A new module version is published to all configured target locations such as template specs, private Bicep registry and Azure DevOps Universal Packages. Published module versions can then be referenced by solutions using them. Further details for this phase are provided on the corresponding wiki page - see the [Publishing](./The%20CI%20environment%20-%20Publishing) page.

   <img src="./media/CIEnvironment/pipelineDesignPhases.png" alt="Pipeline phases" height="200">

### DevOps-Tool-specific design

<details>
<summary>GitHub</summary>

<img src="./media/CIEnvironment/pipelinePhasesGH.png" alt="Pipeline phases GH" height="150">

GitHub workflows map each pipeline phase to a dedicated composite action, to maximize code reusability.
The mapping to the specific composite action is provided below:

| Composite Action | Pipeline phase |
| - | - |
| **validateModulePester** | Static validation |
| **validateModuleDeployment** | Deployment validation |
| **publishModule** | Publishing |

In addition, workflows leverage the following composite actions:

| Composite Action | Description |
| - | - |
| **getWorkflowInput** | This action allows fetching workflow input values from the module's workflow file, even if the pipeline was not triggered via a `workflow_dispatch` action. Without it, we would not be able to process the contained information and would need to duplicate the configuration as workflow variables. Such input values are for example, the removal switch `removeDeployment`. |
| **setEnvironmentVariables** | This action parses the settings file ([`settings.yml`](https://github.com/Azure/ResourceModules/blob/main/settings.yml)) and sets the key-value pairs in the `variables` list as environment variables. |

Technical documentation for each composite action, such as required input and output variables, is included in each `action.yml` file located in path `.github/actions/templates`.

</details>

<details>
<summary>Azure DevOps</summary>

<img src="./media/CIEnvironment/pipelinePhasesADO.png" alt="Pipeline phases ADO" height="300">

Azure DevOps pipelines map each pipeline phase to a dedicated pipeline template, to maximize code reusability.
The mapping to the specific YAML template file is provided below:

| Template Name | Pipeline phase |
| - | - |
| **jobs.validateModulePester.yml** | Static validation |
| **jobs.validateModuleDeployment.yml** | Deployment validation |
| **jobs.publishModule.yml** | Publishing |

Technical documentation for each template, such as required input and output variables, is included in each `.yml` file located in path `.azuredevops/pipelineTemplates`.

</details>

## Module pipeline inputs

Each module pipeline comes with the following runtime parameters:

- `'Branch' dropdown`: A dropdown to select the branch to run the pipeline from.
- `'Remove deployed module' switch`: Can be enabled or disabled and controls whether the test-deployed resources are removed after testing. It is enabled by default.
- `'Publish prerelease module' switch`: Can be enabled or disabled and allows publishing a prerelease version for the corresponding module when running the pipeline from a branch different than `main|master`. It is disabled by default. For further information of how the input is processed refer to the [Publishing](./The%20CI%20environment%20-%20Publishing) dedicated page.

  <img src="./media/CIEnvironment/modulePipelineInput.png" alt="Module Pipeline Input" height="300">

---

# Platform pipelines

In addition to module pipelines, the repository includes several platform pipelines covering further tasks as described below.

- [ReadMe pipeline](#readme-pipeline)
- [Wiki pipeline](#wiki-pipeline)

## ReadMe pipeline

The repository includes two major ReadMe files that should stay in sync with the available modules.

The first can be found in the repository root (`README.md`) and the second in the modules folder (`modules/README.md`).

The ReadMe pipeline is triggered each time changes are pushed to the `main` branch and only if a template in the `modules` folder is being altered. The pipeline leverages the script documented on the [GitHub ReadMe module table update](./The%20CI%20environment%20-%20GitHub%20ReadMe%20module%20table%20update) page.

Once triggered, the pipeline crawls through the library and updates the tables in each corresponding ReadMe file, creating links to the corresponding pipeline runs and updating the list of entries.

## Wiki pipeline

The purpose of the wiki pipeline is to sync any files from the `docs/wiki` folder to the wiki repository. It is triggered each time changes are pushed to the `main` branch and only if files in the `docs/wiki` folder are altered.

> **Note:** Any changes performed directly on the wiki via the UI will be overwritten by this pipeline.
