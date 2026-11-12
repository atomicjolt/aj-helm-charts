# Atomic Jolt Helm Charts

## Charts

| Chart               | Description   |
| ------------------- | ------------- |
| atomic-app          | Ruby on Rails app |
| canvas              | Canvas LMS deployment |
| canvas-rce-api      | Canvas RCE API |
| catalyst            | Elixir web server and worker |
| aa-datatable-engine | Node app used by AA |

The desired container image and other options must be specified in a values files.

## Getting started

Add this repository to helm

```
helm repo add atomicjolt https://atomicjolt.github.io/aj-helm-charts/
```

Install an example

```
helm install app atomicjolt/atomic-app -f values.yaml
```

## atomic-app config files

Starting with atomic-app 3.4.0, application config files can be mounted at `/app/config/k8s` from a
ConfigMap instead of (or in addition to) the `configSecret` Secret. Set `appConfig.enabled: true`
and pass each file as an entry in `appConfig.files`. The chart creates the ConfigMap
`<release>-app-config` (override with `appConfig.configMapName`) before the migrations Job runs,
and rolls the app and worker pods when the contents change. Each key is used as the filename, and
the config files and the Secret must not share any filenames.

With Argo CD, pass the files from git using `fileParameters` (the equivalent of
`helm --set-file`):

```yaml
helm:
  valueFiles:
    - $values/argocd/<app>/<env>/settings.yaml
  fileParameters:
    - name: appConfig.files.app_settings\.yml
      path: $values/argocd/<app>/<env>/config/app_settings.yml
```

Periods in the filename must be escaped with `\.` in the parameter name, as with `--set-file`.

If `appConfig.files` is empty, the chart mounts an existing ConfigMap named
`appConfig.configMapName` instead and does not create it.

## Chart builds

Chart builds are automated by GitHub Actions and are triggered by a merge to main, using the project here: https://atomicjolt.github.io/aj-helm-charts/

### To build a chart manually

Move to the chart directory and update the dependencies:
```
cd chart_name
helm dependencies build .
```

To build the chart file locally
```
cd ../repo
helm package ../chart_name
```

Results in a tar file with the following template `<chart_name>-<chart_version>.tgz`
```
chart_name-1.0.0.tgz
```
