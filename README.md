# flux-helm-test

A repository with example configs for relasing Helm charts via Flux.

# The Helm operator (alpha release)

## Synopsis

The Flux Helm operator provides an extension to Flux
(https://github.com/weaveworks/flux) to be able to automate Helm Chart
releases. In other words, given a desired state as a file in git, it
does `helm install` and `helm upgrade` for you.

A Chart release is described through a
[Kubernetes custom resource](https://kubernetes.io/docs/concepts/api-extension/custom-resources/). Each
custom resource contains all that needs to be known to do a Chart
release. The Flux daemon synchronises these resources from git to the
cluster, and the Flux Helm operator makes sure Helm charts are
released as specified in the resources.

## Custom resource (`FluxHelmRelease`)

To record that you want a chart release, you create a resource of the kind `FluxHelmRelease`.

### Example
```
---
apiVersion: helm.integrations.flux.weave.works/v1alpha2
kind: FluxHelmRelease
metadata:
  name: mongodb
  namespace: test
  labels:
    chart: mongodb
spec:
  chartGitPath: mongodb
  releaseName: mongo-database
  values:
    image: bitnami/mongodb:3.7.1-r1
    imagePullPolicy: IfNotPresent
    persistence:
      enabled: false
```

 - the `name` is mandatory and needs to follow k8s naming conventions
 - the `namespace` is optional. It determines where the resource, and thereby the chart, are created
 - if the `releaseName` field is not provided, the release name used for Helm will be `$namespace-$name`
 - the label `chart` is mandatory, and should match the directory containing the chart
 - `chartGitPath` is the directory containing a chart, given relative to the charts path (provided to the helm-operator)
 - `values` are user customizations of default parameter values from the chart itself

## Prerequisites and limitations

- Tiller (the server component of Helm) should be running in the
  cluster. See
  [the Helm documentation](https://docs.helm.sh/using_helm/#quickstart)
  for how to install Helm. The Helm operator alpha will only deal with
  an unauthenticated Tiller installation (the default).

- You need a git repo with a similar directory structure to this repo
  (if not this repo). That is, the configuration you want sync'ed by
  the Flux daemon.

- The Helm operator alpha only deals with charts in a single git
  repository, which you supply to it as a command-line argument, along
  with a parent directory for charts within that repository. In the
  resources, charts are given relative to the parent directory.

## User setup

We published a step-by-step run-through on how to use Flux and Helm (using
this repository) [over
here](https://github.com/weaveworks/flux/blob/master/site/helm/get-started.md).

## <a name="help"></a>Getting Help

If you have any questions about, feedback for or problems with `flux-helm-test`:

- Invite yourself to the <a href="https://slack.weave.works/" target="_blank">Weave community</a> slack.
- Ask a question on the [#flux](https://weave-community.slack.com/messages/flux/) slack channel.
- Send an email to <a href="mailto:weave-users@weave.works">weave-users@weave.works</a>
- <a href="https://github.com/weaveworks/flux-helm-test/issues/new">File an issue.</a>

Your feedback is always welcome!
