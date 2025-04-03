# Blueprint repo

Blueprint repo for kpt package

```bash
gh auth login
gh repo create blueprint
gh repo create deployment

USER=<YOUR GITHUB USERNAME>
BLUEPRINT_REPO=git@github.com:${USER}/blueprint.git
DEPLOYMENT_REPO=git@github.com:${USER}/deployment.git

git clone ${BLUEPRINT_REPO}
git clone ${DEPLOYMENT_REPO}

cd blueprint

kubectl get pods

cat << 'EOF' > kube-gen.sh
#!/bin/bash
# kube-gen.sh resource-type args
res="${1}"
shift 1
if [[ "${res}" != namespace ]] ; then
  namespace="--namespace=example"
else
  namespace=""
fi
kubectl create "${res}" -o yaml --dry-run=client "${@}" ${namespace} |\
egrep -v "creationTimestamp|status"
EOF
cat kube-gen.sh

chmod a+x kube-gen.sh
sudo mv kube-gen.sh /usr/local/bin
cat /usr/loca/bin/kube-gen.sh
kube-gen.sh --help

mkdir basens
kpt pkg init basens --description "kpt package for provisioning namespace"
kpt pkg tree basens

cd basens
kpt pkg tree

kpt fn eval --type mutator --keywords namespace --image set-namespace:v0.4.1 --fn-config package-context.yaml
kpt fn eval -i set-namespace:v0.4.1 --fn-config package-context.yaml --save -t mutator
cat Kptfile

kpt fn render

kube-gen.sh rolebinding app-admin --clusterrole=app-admin --group=example.admin@bigco.com > rolebinding.yaml
cat rolebinding.yaml

cat > update-rolebinding.yaml << EOF
apiVersion: fn.kpt.dev/v1alpha1
kind: ApplyReplacements
metadata:
  name: update-rolebinding
  annotations:
    config.kubernetes.io/local-config: "true"
replacements:
- source:
    kind: ConfigMap
    name: kptfile.kpt.dev
    fieldPath: data.name
  targets:
  - select:
      name: app-admin
      kind: RoleBinding
    fieldPaths:
    - subjects.[kind=Group].name
    options:
      delimiter: '.'
      index: 0
EOF
cat update-rolebing.yaml

kpt fn eval -i apply-replacements:v0.1.1 --fn-config update-rolebinding.yaml --save -t mutator
kpt fn render

kube-gen.sh quota default --hard=cpu=40,memory=40G > resourcequota.yaml
kpt fn render
cat resourcequota.yaml

kpt pkg tree

cd .. && git add basens && git commit -am "initial pkg"
git push origin main
git tag basens/v0 && git push origin basens/v0
```
