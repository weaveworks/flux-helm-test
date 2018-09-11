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

You can fork this repo and use it as the basis for trying out the Helm
operator alpha. It is best to use a fresh Kubernetes cluster.

 1. Set up helm by running `helm init`. Additional configuration may be
    required when using [GKE](https://docs.helm.sh/using_helm/#gke) or
    [other Kubernetes Distributions](https://docs.helm.sh/using_helm/#kubernetes-distribution-guide).
    Have a read through
    [Helm's installation instructions](https://docs.helm.sh/using_helm/#quickstart).
 2. Make a fork of this repository to your own account, and clone it to your computer
 3. In your cloned repo, change the git URL in
    `artifacts/weave-helm-operator.yaml` to refer to your fork on github. Make
    sure to use the full ssh form of the url.
    - Good: `ssh://git@github.com/weaveworks/flux-helm-test`
    - Bad: `git@github.com:weaveworks/flux-helm-test.git`
 4. Go to [Weave Cloud](https://cloud.weave.works/) and create an instance
 5. Set up the Weave agents by running the provided curl command

 6. Before configuring "Deploy" for your new Weave Cloud instance, run
    this in the cloned repository to install the Helm operator:

    kubectl apply -f artifacts/

 7. Now go back to Weave Cloud and set "Deploy" up, giving the git URL
    of your forked repo. You will need to
    install the deploy key (either using the button, or by copying and
    pasting), since both the operator and the Flux daemon need it to
    access your forked git repository. Run the kubectl command given
    there to complete the setup.

At this point, you can click through to Explore, wait a bit, and see
the resources started in the cluster by the charts being released. If
you make a change in the chart, or a change in the chart release
resource, and push a commit to your fork, the change will be reflected
shortly in your cluster.

## <a name="help"></a>Getting Help

If you have any questions about, feedback for or problems with `flux-helm-test`:

- Invite yourself to the <a href="https://slack.weave.works/" target="_blank">Weave community</a> slack.
- Ask a question on the [#flux](https://weave-community.slack.com/messages/flux/) slack channel.
- Send an email to <a href="mailto:weave-users@weave.works">weave-users@weave.works</a>
- <a href="https://github.com/weaveworks/flux-helm-test/issues/new">File an issue.</a>

Your feedback is always welcome!
