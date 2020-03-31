# ocp-service-mesh-rbac-test

* Stand up the OpenTLC OCP 4 Service Mesh Lab (1master + 2workers)

* SSH into the clientvm with the provided credentials

```
mkdir gitrepos
cd gitrepos
```

* clone 1

```
git clone https://github.com/raffaelespazzoli/openshift-enablement-exam.git
```

* clone 2

```
git clone https://github.com/trevorbox/bookinfo.git
```

* Create htpasswd accounts in cluster

Notes: The OpenTLC provisioning of the cluster already created
the admin account with cluster-admin role.  We need to add it
back into htpasswd when that file is created.

```
cd openshift-enablement-exam/misc4.0/htpasswd/
```

```
oc whoami
```

* Above command should return system:admin

```
export MYUSER=worsco
htpasswd -c -B -b htpasswd $MYUSER $MYUSER
htpasswd -B -b htpasswd admin secretPASSWORD
oc create secret generic htpass-secret --from-file=htpasswd=htpasswd -n openshift-config
oc apply -f oauth.yaml -n openshift-config
```

```
oc get route console -n openshift-console -o jsonpath='{.spec.host}' ; echo ""
```

* Install the Service Mesh

```
cd ~/gitrepos/openshift-enablement-exam/misc4.0/ServiceMesh
```

```
oc new-project istio-system
oc apply -f operators.yaml
# Pause here
oc apply -f control_plane.yaml
```

https://docs.openshift.com/container-platform/4.3/applications/projects/creating-project-other-user.html

```
oc new-project bookinfo --as=$MYUSER --as-group=system:authenticated --as-group=system:authenticated:oauth
```

```
oc apply -f servicememberroll.yaml -n istio-system
oc apply -n bookinfo -f https://raw.githubusercontent.com/Maistra/bookinfo/maistra-1.0/bookinfo.yaml
oc apply -n bookinfo -f https://raw.githubusercontent.com/Maistra/bookinfo/maistra-1.0/bookinfo-gateway.yaml
export istio_gateway_url=$(oc get route istio-ingressgateway -n istio-system -o jsonpath='{.spec.host}')
curl http://$istio_gateway_url/productpage
```

```
cd ~/gitrepos/bookinfo/.service-mesh/service-mesh/rbac-test/
```

```
export KIALI_URL=$(oc get route jaeger -n istio-system -o jsonpath='{.spec.host}')
echo "https://$KIALI_URL/"
```

* Log into Kiali via a web browser.

```
cd ~/gitrepos/bookinfo/.service-mesh/service-mesh/rbac-test
```

```
find . -name '*.yml' -type f -printf '\n%p:\n' -exec sed -i "/tbox/{
s//$MYUSER/g
w /dev/stdout
}" {} \;
```

```
oc apply -f 01-cluster-roles.yml
oc apply -f 02-bookinfo-role-bindings.yml
oc apply -f 03-istio-system-role-bindings.yml
```

* Log into Jaeger. Retrieve route name and log into it via browser https://

```
export JAEGER_URL=$(oc get route jaeger -n istio-system -o jsonpath='{.spec.host}')
echo "http://$JAEGER_URL/"
```

* Log into Kiali via browser...
```
export KIALI_URL=$(oc get route jaeger -n istio-system -o jsonpath='{.spec.host}')
echo "https://$KIALI_URL/"
```

* Log into Grafana via browser...

```
export GRAFANA_URL=$(oc get route grafana -n istio-system -o jsonpath='{.spec.host}')
echo "https://$GRAFANA_URL/"
```
