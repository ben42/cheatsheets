# kustomize

#### Applying labels in CustomResources

openshift-config.yaml
```
# Used to enable commonLabelTransformer for specific path in crd
commonLabels:
- kind: DeploymentConfig
  path: spec/template/metadata/labels
  # creates path if not present
  create: true
```

kustomization.yaml
```
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

commonLabels:
  app.kubernetes.io/managed-by: cicd

configurations:
- kustomizeconfig/openshift-config.yaml

resources:
- deploymentconfigs.yaml
```

#### Referencing `vars` in CustomResources 

deploymentconfigs.yaml
```
apiVersion: v1
kind: DeploymentConfig
metadata:
  name: wekan
spec:
  template:
    spec:
      containers:
      - name: wekan
        env:
        - name: ROOT_URL
          value: $(WEKAN_ROOT_URL)
```

routes.yaml
```
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  name: wekan
spec:
  host: wekancicd-test-kustomize.ocp.company.org
  port:
    targetPort: wekan
  tls:
    insecureEdgeTerminationPolicy: Redirect
    termination: edge
  to:
    kind: Service
    name: wekan
```

kustomization.yaml
```
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
- routes.yaml
- deploymentconfigs.yaml

configurations:
- kustomizeconfig/openshift-config.yaml

vars:
  - name: WEKAN_ROOT_URL
    objref:
      apiVersion: route.openshift.io/v1
      kind: Route
      name: wekan
    fieldref:
      fieldpath: spec.host
```

openshift-config.yaml
```
# mandatory because the resource DeploymentConfig is
# not covered by the default transformer settings
varReference:
- kind: DeploymentConfig
  path: spec/template/spec/containers/env/value
```

#### Dump transformers configuration
```
kustomize config save -d ./tmp
```

#### Links

* https://kubectl.docs.kubernetes.io/pages/examples/kustomize.html
