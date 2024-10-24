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
