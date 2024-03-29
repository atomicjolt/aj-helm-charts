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
helm repo add atomicjolt https://atomicjolt-helm-charts.s3.amazonaws.com
```

Install an example

```
helm install app atomicjolt/atomic-app -f values.yaml
```

## To build a chart

Move to the chart directory and update the dependencies:
```
cd chart_name
helm dependencies build .
```

Then build the chart file
```
cd ../repo
helm package ../chart_name
curl -O https://atomicjolt-helm-charts.s3.amazonaws.com/index.yaml
helm repo index --merge index.yaml .
aws s3 cp index.yaml s3://atomicjolt-helm-charts/
aws s3 cp chart_name-1.0.0.tgz s3://atomicjolt-helm-charts/
```
