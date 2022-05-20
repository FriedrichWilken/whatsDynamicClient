F' this, let's mess around with CRDs by building dynamic clients. Why? To prevent the dependency hell of using the actual defintion of the CRD by importing a whole repo of stuff you don't need and don't want. That's why.

First, we need a cluster with some shiny CRDs. I will use [k3d](https://k3d.io) with [kyma](https://kyma-project.io). Kyma is batteries included Kubernetes with fancy stuff (eventing, serverless, tracing/logging/monitoring, ...) that get's in there via CRDs. Nice, just what we need. 

You can get both on mac with brew:
```shell
brew install k3d kyma-cli
```
or via choco on win:
```shell
choco install k3d kyma-cli
```
here are more instruction for the installation on all OS of [k3d](https://k3d.io/v5.4.1/#installation) and [kyma-cli](https://kyma-project.io/docs/kyma/latest/04-operation-guides/operations/01-install-kyma-CLI/).

Next, let's provision a cluster and deploy kyma on it:
```shell
kyma provision k3d && kyma deploy
```
then we need a namespace
```shell
kubectl create namespace my-mess
```
and then we create an subscription in it
```shell
cat << EOF | kubectl apply -f -
apiVersion: eventing.kyma-project.io/v1alpha1
kind: Subscription
metadata:
  name: messinaround
  namespace: my-mess
spec:
  filter:
    filters:
    - eventSource:
        property: source
        type: exact
        value: ""
      eventType:
        property: type
        type: exact
        value: sap.kyma.custom.noapp.order.created.v1
  protocol: ""
  protocolsettings: {}
  sink: http://test.my-mess.svc.cluster.local
EOF
```
we don't really need to know what a `subscription` for this little experiment (but if you are interested, you can learn more about kymas shiny eventing system [here](https://kyma-project.io/docs/kyma/latest/05-technical-reference/00-architecture/evnt-01-architecture/))

Now, have a look on what we created:
```shell
kubectl get subscriptions -n my-mess -oyaml
```

todo
