# OpenShift Install

[![IaC](https://app.soluble.cloud/api/v1/public/badges/ff30d9bc-e72e-405d-ab72-af397117e392.svg)](https://app.soluble.cloud/repos/details/github.com/ryhennessy/openshiftv4-install)  [![CIS](https://app.soluble.cloud/api/v1/public/badges/81d3d726-f94d-4948-a2a7-2f329c1f27bf.svg)](https://app.soluble.cloud/repos/details/github.com/ryhennessy/openshiftv4-install)  

* source .credentials.aws.sh (AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY env vars)

* download `openshift-install` binary and run ```./openshift-install create install-config```

* get secret from the openshift website

```shell
export KUBECONFIG=$(pwd)/auth/kubeconfig
```

* install helm

```shell
oc new-project helm
oc apply -f helm-rbac.yaml
helm init --service-account helm --tiller-namespace helm
helm list --tiller-namespace helm
```

* create dedicated namespace/project for datadog
```shell
oc new-project datadog
oc apply -f install/scc.yaml
oc get scc datadog-agent

# update the `install/values.yaml file with your datadog credentials`

helm install --tiller-namespace helm --name datadog -f install/values.yaml stable/datadog
oc apply -f install/clusterrolebinding-agent.yaml
```

* deploy control-plan monitoring

```shell
# install the "pause pods" on each master node, this is the trick to get around the missing static pod status in kubelet pod list.
oc apply -f install/daemonset-master-pause.yaml

# create services on top of the control plan components in order to schedule the Datadog endpoint checks 
oc apply -f install/datadog-api-server-metrics_svc.yaml
oc apply -f install/datadog-controller-manager-metrics_svc.yaml
oc apply -f install/datadog-scheduler-metrics_svc.yaml
```
