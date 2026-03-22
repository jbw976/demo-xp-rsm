# Demo - Crossplane Resource State Metrics

This demo walks through some demo usage of https://github.com/crossplane-contrib/resource-state-metrics.

## Pre-reqs

Clone the https://github.com/crossplane-contrib/resource-state-metrics repo locally:
```
git clone https://github.com/crossplane-contrib/resource-state-metrics.git
cd resource-state-metrics
```

## Create resources

Start the repo with `make local` which will create all the essential
infrastructure, install the resource state metrics configuration package, create
multiple EKS clusters, and start monitoring everything with
`ResourceMetricsMonitors`.

This pulls/downloads a lot of packages and containers, so it's wise to do this
and let it finish before using conference wifi.

```
export AWS_CLOUD_CREDENTIALS="$(cat ~/.aws/credentials)"
make local
```

After everything has been set up, start a port forward to the grafana server:
```
kubectl port-forward -n monitoring svc/kube-prometheus-stack-grafana 3000:80
```

Go to the Grafana dashboard at http://localhost:3000 and loging using these credentials:
```
admin
echo $(kubectl get secret --namespace monitoring -l app.kubernetes.io/component=admin-secret -o jsonpath="{.items[0].data.admin-password}" | base64 --decode)
```

## Examine the dashboard
From the Grafana dashboards list, open the `Crossplane Resource State Metrics` dashboard. There are many interesting metrics to browse, like:

* Health percentage for your XRs, Providers, Functions, etc.
* EKS Ready % over time
* EKS XR responsive status (circuit breaker)
* Time in not ready state
* Cardinality counts and limits
* etc.

## `ResourceMetricsMonitor` objects

Look at all the `ResourceMetricsMonitor` objects that have been defined on the control plane, actively scraping metrics:
```
kubectl -n resource-state-metrics get ResourceMetricsMonitor
```

## Provider Health example
Let's look at the implementation of a specific (and simple) ResourceMetricsMonitor:

https://github.com/crossplane-contrib/resource-state-metrics/blob/main/examples/metrics/provider/healthy.yaml


## Cardinality limits

We can define limits on cardinality for our monitors and get information about the cardinality in real time:
```
kubectl -n resource-state-metrics get ResourceMetricsMonitor provider-health -o json | jq '.status.cardinality'
```

Here's an example of a cardinality limit being set for one of our monitors:

https://github.com/crossplane-contrib/resource-state-metrics/blob/main/examples/metrics/eks/cardinality-limits.yaml

## Parent correlation

We can monitor metadata on each object to then use as information to connect parent XRs and their child composed resources:

https://github.com/crossplane-contrib/resource-state-metrics/blob/main/examples/metrics/eks/mr-parent-correlation.yaml


## Tests

We can also write test cases for our monitoring, to make sure it's discovering the results that we expect:

https://github.com/crossplane-contrib/resource-state-metrics/blob/main/promql-tests/eks/mr-parent-correlation_test.yaml

# Clean-up

When you're done with the demo, clean up all the infrastructure resources and monitoring with:
```
make local-clean
```