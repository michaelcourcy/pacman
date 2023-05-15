# Work with Azure Devops

Demonstrate how you can insert a Kasten backup in the middle of a CD process with Azuredevops 

# fork the project in github 

In order to experiment fork the project https://github.com/michaelcourcy/pacman

# prepare the different service connections 

## Create a connection to your cluster 

Connect to your cluster and create a pacman ns, a service account with appropriate rolebinding. 
Create also the secret token for this sa.

```
kubectl create ns pacman
kubectl apply -f -<<EOF
apiVersion: v1
kind: ServiceAccount
metadata:
   name: devops
   namespace: pacman 
EOF
kubectl apply -f -<<EOF
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: role-for-devops
  namespace: pacman
rules:
- apiGroups: ["*","apps","extensions"]
  resources: ["*"]  
  verbs: ["*"]
EOF
kubectl apply -f -<<EOF
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: rolebinding-for-devops
  namespace: pacman
subjects:
- kind: ServiceAccount
  name: devops
  namespace: pacman
roleRef:
  kind: Role
  name: role-for-devops
  apiGroup: rbac.authorization.k8s.io
EOF
kubectl apply -f -<<EOF
apiVersion: v1
kind: Secret
type: kubernetes.io/service-account-token
metadata:
  name: sa-secret
  namespace: pacman
  annotations:
      kubernetes.io/service-account.name: "devops"
EOF
```

```
k create clusterrole view-on-kasten-io-basic --verb=get,list,watch --resource=clusterroles --resource-name=kasten-io-basic
k create clusterrolebinding devops-kasten-io-basic --clusterrole=view-on-kasten-io-basic --serviceaccount=pacman:devops

kubectl create clusterrole kasten-io-basic --as system:serviceaccount:pacman:devops

kubectl create rolebinding kasten-io-basic-pac-backupaction --namespace=pacman --clusterrole=kasten-io-basic  --serviceaccount=pacman:pac-backupaction  --as system:serviceaccount:pacman:devops
```



Go in project settings > Pipelines > Service connections and create a `Kubernetes` connection. fill up the form with this information 
- Service connection name : you choose
- namespace : pacman
- server url : `kubectl config view --minify -o jsonpath='{.clusters[0].cluster.server}'`
- secret : `kubectl get secret -n pacman sa-secret -o json`
- accept untrusted certificate : true

Note the name of the 

## Create a registry connection for pushing your image 

Go in project settings > Pipelines > Service connections and create a `docker registry` connection.

## Create a github repo 

Go in project settings > Pipelines > Service connections and create a `github` connection

# Create the pipeline

In the source code make sure the section variable is consistent with the different name your give to your service connections.

If not fix and push to your fork.

Go in Pipelines > pipelines and  Create Pipeline with your guthub connection. Azure-pipelines.yaml should be automatically loaded.



