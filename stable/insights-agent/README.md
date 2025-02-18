# Fairwinds Insights Agent

The Fairwinds Insights Agent is a collection of reporting tools, which send data back
to [Fairwinds Insights](https://insights.fairwinds.com).

## Installation
We recommend installing `insights-agent` in its own namespace.

```
helm repo add fairwinds-stable https://charts.fairwinds.com/stable
helm upgrade --install insights-agent fairwinds-stable/insights-agent \
  --namespace insights-agent \
  --set insights.organization=acme-co \
  --set insights.cluster=staging \
  --set insights.base64token="abcde=="
```

## CronJob schedules
Most Insights reports run as CronJobs, typically every hour.

In order to avoid every report consuming resources at once, you can stagger the reports
by setting the minute of the cron schedule to a random number. For example,
to run at a random minute every hour, your cron expression can be `rand * * * *`.

## Reports
There are several different report types which can be enabled and configured:
* `polaris`
* `goldilocks`
* `workloads`
* `kubehunter`
* `trivy`
* `kubesec`
* `nova`
* `rbacreporter`
* `kubebench`
* `pluto`
* `opa`
* `resourcemetrics`
* `admission`
* `awscosts`

See below for configuration details.

## Fleet Installation
If you're installing the Insights Agent across a large fleet of clusters,
it can be tedious to use the UI to create each cluster, then copy out the
cluster's access token.

To better serve customers with a large number of clusters, we've created a flow
that allows you to easily deploy the Insights Agent across your fleet.
[Read more here](https://insights.docs.fairwinds.com/run/agent/installation/#fleet-installation)

## Configuration
Parameter | Description | Default
--------- | ----------- | -------
`insights.organization` | The name of the organization to upload data to | ""
`insights.cluster` | The name of the cluster the data is coming from | ""
`insights.base64token` | Your cluster's base64-encoded auth token provided by Insights | ""
`insights.tokenSecretName` | If you don't provide a `base64token`, you can specify the name of a secret to pull the token from | ""
`insights.host` | The location of the Insights server | https://insights.fairwinds.com
`rbac.disabled` | Don't use any of the built-in RBAC | `false`
`fleetInstall` | See Fleet Installation docs | `false`
`insights.apiToken` | Only needed if `fleetInstall=true` | ""
`uploader.image.repository`  | The repository to pull the uploader script from | quay.io/fairwinds/insights-uploader
`uploader.image.tag` | The tag to use for the uploader script | 0.2
`uploader.resources` | CPU/memory requests and limits for the uploader script |
`uploader.sendFailures` | Send logs of failure to Insights when a job fails. | true
`uploader.env` | Set extra environment variables for the uploader script | []
`cronjobs.disableServiceMesh` | Adds annotations to all CronJobs to not inject Linkerd or Istio | true
`cronjobs.backoffLimit` | Backoff limit to use for each report CronJob | 1
`cronjobs.failedJobsHistoryLimit` | Number of failed jobs to keep in history for each report | 2
`cronjobs.successfulJobsHistoryLimit` | Number of successful jobs to keep in history for each report | 2
`cronjobs.nodeSelector` | Node selector to use for cronjobs | null
`cronjobs.tolerations` | Tolerations to use for cronjobs | null
`cronjobs.runJobsImmediately` | Run each of the reports immediately upon install of the Insights Agent | true
`cronjobs.dnsPolicy` | Adds pod DNS policy |
`cronjobs.imagePullSecret` | Name of a pull secret to attach to all CronJobs |
`{report}.enabled` | Enable the report type |
`{report}.schedule` | Cron expression for running the report | `rand * * * *`
`{report}.timeout` | Maximum time in seconds to wait for the report |
`{report}.resources` | CPU/memory requests and limits for the report |
`{report}.image.repository` | Repository to use for the report image |
`{report}.image.tag` | Image tag to use for the report |
`polaris.config` | A custom [polaris configuration](https://polaris.docs.fairwinds.com/customization/configuration/)
`polaris.extraArgs` | A string of custom arguments to pass to the polaris CLI, e.g. `--disallow-annotation-exemptions=true` | 
`kubehunter.logLevel` | DEFAULT, INFO, or WARNING | `INFO`
`kubebench.mode` | Changes the way this plugin is deployed, either `cronjob` where it will run a single pod on the `schedule` that will pull the data necessary from a single node and report that back to Insights. `daemonset` which deploys a daemonset to the cluster which gathers data then a cronjob will gather data from each of those pods. `daemonsetMaster`  is the same as `daemonset` except the daemonset can also run on masters. | `cronjob`
`kubebench.hourInterval` | If running in `daemonset` or `daemonsetMaster` this configuration changes how often the daemonset pods will rescan the node they are running on | 2
`kubebench.aggregator` | contains `resources` and `image.repository` and `image.tag`, this controls the pod scheduled via a CronJob that aggregates from the daemonset in `daemonset` or `daemonsetMaster` deployment modes. |
`trivy.privateImages.dockerConfigSecret` | Name of a secret containing a docker `config.json` | ""
`trivy.maxConcurrentScans` | Maximum number of scans to run concurrently | 1
`trivy.maxScansPerRun` | Maximum number of images to scan on a single run | 20
`trivy.namespaceBlacklist` | Specifies which namespaces to not scan, takes an array of namespaces for example: `--set trivy.namespaceBlacklist="{kube-system,default}"` | nil
`opa.role` | Specifies which ClusterRole to grant the OPA agent access to | view
`opa.additionalAccess` | Specifies additional access to grant the OPA agent. This should contain an array of objects with each having an array of apiGroups, an array of resources, and an array of verbs. Just like a RoleBinding. | null
`opa.installCRDs` | Specifies whether to install the `customcheckinstances.insights.fairwinds.com` CRD. If you are installing the `insights-agent` chart twice you will want to set this flag to `false` on *one* of the installs, doesn't matter which. | true
`goldilocks.controller.flags.exclude-namespaces` | Namespaces to exclude from the goldilocks report | `kube-system`
`goldilocks.vpa.enabled` | Install the Vertical Pod Autoscaler as part of the Goldilocks installation | true
`goldilocks.controller.flags.on-by-default` | Goldilocks will by default monitor all namespaces that aren't excluded | true
`goldilocks.controller.resources` | CPU/memory requests and limits for the Goldilcoks controller |
`goldilocks.dashboard.enabled` | Installs the Goldilocks Dashboard | false
`resourcemetrics.installPrometheus` | Install a new Prometheus instance for the resourcemetrics report | false
`resourcemetrics.address` | The address of an existing Prometheus instance to query in the form `<scheme>://<service-name>.<namespace>[:<port>]` for example `http://prometheus-server.prometheus` | `"http://prometheus-server"`
`nova.logLevel` | The klog log-level to use when running Nova | `3`
`pluto.targetVersions` | The versions to target, e.g. `k8s=1.21.0` | Defaults to current Kubernetes version
`awscosts.secretName` | Kubernetes Secret name where AWS creds will be stored | ""
`awscosts.awsAccessKeyId` | AWS access Key ID for AWS costs | ""
`awscosts.awsSecretAccessKey` | AWS access key secrect for AWS costs | ""
`awscosts.region` | AWS region where costs was defined | ""
`awscosts.database` | AWS database where Athena table was created | ""
`awscosts.table` | AWS database Athena table for AWS costs | ""
`awscosts.catalog` | AWS database catalog for AWS costs | ""
`awscosts.tagkey` | Tag used to identify cluster nodes. Example: Kops uses 'kubernetes_cluster'.  | ""
`awscosts.tagvalue` | Tag value used to identify a cluster given a tag key. | ""
`awscosts.workgroup` | Athena work group that used to run the queries | ""
