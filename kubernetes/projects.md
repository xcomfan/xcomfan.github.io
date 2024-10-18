---
layout: page
title: "Kubernetes: Organizing Your Application"
permalink: /kubernetes/projects
---

## Principles to Guide Us

* File systems as the source of truth
* Code review to ensure the quality of changes
* Feature flags for staged roll forward and roll back

## Feature Gates and Guard

Feature gates and guards play an important role in bridging the gap of developing new features in source control and the deployment of those features into production. You can do the development behind a feature flag or gate as in the sippet below. This enables committing of code to the production branch long before the feature is ready to ship. The enabling of the feature requires just a configuration change to activate the flag. This makes it very clear what changed in the production environment, and makes it easy to roll back without having to go back to an older version of the code.
 
```javascript
if (featureFlags.myFlag){
  // Feature implementation goes here
}
```

## Filesystem layout

The first cardinality on which you want to organize your application is the semantic component or layer (e.g., frontend, batch work queue etc.) This may seem like overkill when one team manages all these components but it makes it easy to scale the team later. An application with a front end that uses two services would have a file layout similar to the below. Within each directory the configuration for each application is stored. These are YAML files that directly represent the current state of the cluster. Its generally useful to include both service name and the object type withing the same file. While Kubernetes allows for the creation of YAML files with multiple objects in the same file, this should generally be avoided. The only good reason to group objects in the same file is if they are conceptually identical. If grouping objects together doesn't form a single concept, they probably shouldn't be in a single file.

```text
frontend/
  frontend-deployment.yaml
  frontend-service.yaml
  frontend-ingress.yaml
service-1/
  service-1-deployment.yaml
  service-1-service.yaml
  service-1-configmap.yaml
service-2/
  ...
```

## Managing periodic versions

It is very useful to be able to look back historically and see what your application deployment previously looked like. It is also useful to be able to iterate a configuration forward while still being able to deploy a stable release configuration. Thus its handy to be able to simultaneously store and maintain multiple different revisions of your configuration. There are two approaches to this.

The first is to use tags, branches and source-control features. This leads to a more simplified directory structure and aligns with how source code revisions are managed. The second options is to clone the configuration within the filesystem and use directories for different revisions. This approach is convenient because it makes simultaneous viewing of the configuration very straightforward. Both of these are essentially identical and are just a matter of preference.

### Versioning With Branches and Tags

When you are ready for release, you place a source-control tag (e.g `git tag v1.0`) on the config version and the HEAD continues to iterate forward. When you need to update the release configuration; first you commit the change to HEAD of the repository, then you create a new branch named v1 at the v1.0 tag. You then cherry-pick the desired change onto the release branch (`git cherry-pick <edit>`), and finally you tag this branch with the `v1.1` tag to indicate the new point release. ***Note:*** A common error is to cherry pick fixes into a release branch only. Its a good idea to cherry-pick it into all active releases, in case for some reason you need to roll back versions but the fix is still needed.

[comment]: <> (TODO: Work though this to see how it actually works)

### Versioning With Directories

In this approach your versioned deployment exists within its own directory as in the example below. All deployments occur from `HEAD` instead of from specific revision tags. When adding a new configuration it is done to the files in the `current` directory. When creating a new release the current directory is copied to create a new directory associated wit the new release. When performing a bugfix change to a release, the pull request must modify the YAML file in all the relevant release directories. This is a slightly better experience than the cherry-picking approach described earlier, since it is clear in a single change request that all of the relevant versions are being updated with the same change, instead of requiring a cherry-pick per version.

```text
frontend/
  v1/
    frontend-deployment.yaml
    frontend-service.yaml
  current/
    frontend-deployment.yaml
    frontend-service.yaml
service-1/
  v1/
    service-1-deployment.yaml
    service-1-service.yaml
  v2/
    service-1-deployment.yaml
    service-1-service.yaml
  current/
    service-1-deployment.yaml
    service-1-service.yaml
...
```

## Structuring Your Application For Development, Testing and Deployment

There are two goals for your application with regard to development and testing.

* Each developer should be able to easily develop new features for the application. Developers should be able to work in their own environment with all services available.

* You should be able to easily and accurately test your application prior to deployment. This is essential to being able to quickly roll out features while maintaining availability.

### Progression Of a Release

To achieve both these goals it is important to relate stages of development to the release versions described earlier. The stages of release are:

* HEAD - The bleeding edge of the configuration; the latest changes

* Development - Largely stable, but not ready for deployment. Suitable for developers to use for building features.

* Staging - The beginning of testing, unlikely to change unless problems are found.

* Canary - The first release to users, used to test for problems with real world traffic and likewise give user a chance to test what is coming next.

* Release - the current production release

You should use a tag to mark the development stage. An automated process should be used to test the `HEAD` branch, and if test pass the `development` tag is moved to `HEAD`. Thus developers can track reasonably close to the latest changes when deploying their own environments.

To map the stage to a specific revision if using source control approach you would use tags and if using file system approach you would use symbolic links.

## Parameterizing Your Applications With Templates

Since its impractical to keep the development stages identical, but you want the environments to be as identical as possible its a good idea to have parameterized environments with templates. This lets you use a template for the bulk of the configuration but have a limited set of parameters to produce the final configuration. This makes it easy to see the differences between environments.

### Parameterizing With Helm and Templates

[Helm](https://helm.sh) is a package manager for Kubernetes. They patterns described here for Helm will apply to whatever templating option you choose.

Helm templating language use the "mustache" syntax, so for example

```yaml
metadata:
  name: {{ .Release.Name }}-deployment
```

indicates that `Release.Name` should be substituted into the name of a deployment. To pass a parameter for this value you use `values.yaml` file with contents like the below.

```yaml
Release:
  Name: my-release
```

After parameter substitution you would get ...

```yaml
metadata:
  name: my-release-deployment
```

### Filesystem Layout For Parameterization

With parametarization, instead of treating each deployment lifecycle stage as a pointer to a version, each deployment lifecycle is the combination of a parameter file and a pointer to a specific version.  Below is an example of such a layout.  

```text
frontend/
  staging/
    templates -> ../v2
    staging-parameters.yaml
  production/
    templates -> ../v1
    production-parameters.yaml
  v1/
    frontend-deployment.yaml
    frontend-service.yaml
  v2/
    frontend-deployment.yaml
    frontend-service.yaml
```

Doing this with version control looks similar, except that the parameters for each life-cycle stage are kept at the root of the configuration directory tree.

```text
frontend/
  staging-parameters.yaml
  templates/
  frontend-deployment.YAML
```

Should you need to have different configurations for different regions, your directory layout would lool like the below.

```text
frontend/
  staging/
    templates -> ../v3/
    parameters.yaml
  eastus/
    templates -> ../v1/
    parameters.yaml
  westus/
    templates -> ../v2/
    parameters.yaml
...
```

If using version control your layout would look like...

```text
frontend/
  staging-parameters.yaml
  eastus-parameters.yaml
  westus-parameters.yaml
  templates/
    frontend-deployment.yaml
...
```