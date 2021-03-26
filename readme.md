

# Login to your cluster

- Browse to the OpenShift console. From the dropdown menu in the upper right of the page, click Copy Login Command. Paste the copied command in your local terminal.

- Verify you can communicate with your cluster.
```
oc version
```

# Install istio

```
oc adm policy add-scc-to-group anyuid system:serviceaccounts:istio-system
istioctl install --set profile=openshift
oc -n istio-system expose svc/istio-ingressgateway --port=http2
```


# Create namespaces

```
kubectl create namespace travel-agency
kubectl create namespace travel-portal
kubectl create namespace travel-control
```

# Give priviliges to istio and create cni in each namespace
```
oc adm policy add-scc-to-group privileged system:serviceaccounts:travel-agency
oc adm policy add-scc-to-group anyuid system:serviceaccounts:travel-agency
cat <<EOF | oc -n travel-agency create -f -
apiVersion: "k8s.cni.cncf.io/v1"
kind: NetworkAttachmentDefinition
metadata:
  name: istio-cni
EOF

oc adm policy add-scc-to-group privileged system:serviceaccounts:travel-portal
oc adm policy add-scc-to-group anyuid system:serviceaccounts:travel-portal
cat <<EOF | oc -n travel-portal create -f -
apiVersion: "k8s.cni.cncf.io/v1"
kind: NetworkAttachmentDefinition
metadata:
  name: istio-cni
EOF

oc adm policy add-scc-to-group privileged system:serviceaccounts:travel-control
oc adm policy add-scc-to-group anyuid system:serviceaccounts:travel-control
cat <<EOF | oc -n travel-control create -f -
apiVersion: "k8s.cni.cncf.io/v1"
kind: NetworkAttachmentDefinition
metadata:
  name: istio-cni
EOF
```

# Deploy application 

```
kubectl apply -f <(curl -L https://raw.githubusercontent.com/kiali/demos/master/travels/travel_agency.yaml) -n travel-agency
kubectl apply -f <(curl -L https://raw.githubusercontent.com/kiali/demos/master/travels/travel_portal.yaml) -n travel-portal
kubectl apply -f <(curl -L https://raw.githubusercontent.com/kiali/demos/master/travels/travel_control.yaml) -n travel-control
```

# Wait till all deployments are rolled out as expected

```
kubectl get deployments -n travel-agency
kubectl get deployments -n travel-portal
kubectl get deployments -n travel-control
```