# Deploying the DaemonSet and CronJob

The ToDo app (Deployment + ClusterIP Service), the DaemonSet and the CronJob all run in
the `mateapp` namespace. The DaemonSet and CronJob reach the app through the
`todoapp-service` ClusterIP service.

## Deploy

1. Create the namespace and deploy the ToDo app with its ClusterIP service:

   ```
   kubectl apply -f .infrastructure/namespace.yml
   kubectl apply -f .infrastructure/deployment.yml
   kubectl apply -f .infrastructure/clusterIp.yml
   ```

2. Deploy the DaemonSet and the CronJob:

   ```
   kubectl apply -f .infrastructure/daemonset.yml
   kubectl apply -f .infrastructure/cronjob.yml
   ```

## Validate

1. Check that the app is up and the objects exist:

   ```
   kubectl get pods,svc -n mateapp
   kubectl get daemonset,cronjob -n mateapp
   kubectl get pods -n mateapp
   ```

   The DaemonSet should have one pod per node in `Running` state.

2. DaemonSet logs (every 5 seconds the pod curls the app and prints the HTML response):

   ```
   kubectl logs -n mateapp daemonset/busybox --tail=50
   ```

   To follow the logs live:

   ```
   kubectl logs -n mateapp -l app=busybox -f
   ```

3. CronJob: it runs every 4 minutes. To avoid waiting, trigger a run manually:

   ```
   kubectl create job --from=cronjob/curl-cronjob manual-test -n mateapp
   ```

   Otherwise wait for the scheduled run and list the jobs:

   ```
   kubectl get jobs -n mateapp
   ```

   Read the logs of a job (the health endpoint response is expected):

   ```
   kubectl logs -n mateapp job/<job-name>
   ```

   Also check the history limits and concurrency policy:

   ```
   kubectl get cronjob curl-cronjob -n mateapp -o jsonpath='{.spec.successfulJobsHistoryLimit} {.spec.failedJobsHistoryLimit} {.spec.concurrencyPolicy}'
   ```

   Expected output: `10 5 Allow`.

## Cleanup

```
kubectl delete -f .infrastructure/cronjob.yml -f .infrastructure/daemonset.yml
kubectl delete namespace mateapp
```
