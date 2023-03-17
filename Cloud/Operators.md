# **Operators**

**Note:**
* This lab exercise was performed on Macbook. 
* As prerequisite Docker is already installed.
* Operators are software extension to Kubernetes that make use of custom resources to manage applications and their components. Minikube is installed to mimic Kubernetes environment to install and test the operator.
* Helm, Ansible or Go can be used as Operator SDK. Ansible is used in this tutorial.
* A demo web application is already built and containeraized. The image is available in centraltechhub/sessionwebapp:v1. Users are free to make use of this container image or use one of their own.
* The aim of this document is only to show a demo on how to build a simple operator, not to teach the theoretical concept of Operator or it's related technology. To learn more about operators, refer to the official site: https://kubernetes.io/docs/concepts/extend-kubernetes/operator/
 


**1. Create a new minikube instance with 4GB memory and 2 CPUs.**

Execute the following command:
```CMD
minikube start --memory 4096 cpus 2
```

output:
![image](https://user-images.githubusercontent.com/93929892/196854997-6a5dea30-38b2-414b-80ed-45309ec04416.png)

run the following command to check if all the relevant pods are up and running.
```CMD
kubectl get pods --all-namespaces
```

![image](https://user-images.githubusercontent.com/93929892/196855249-94e1e9fc-e528-4b48-922a-4b6688309fdf.png)

**2. Install Operator Lifecycle Manager**

Operator Lifecycle Manage (OLM) is a component of the Operator Framework, an open source toolkit to manage Operators, in a streamlined and scalable way.

Execute the following command:
```CMD
operator-sdk olm install
```

output:
![image](https://user-images.githubusercontent.com/93929892/196855953-8305cdd0-a220-4a59-87c5-82ffca129732.png)

Sample review of a particular kind.

![image](https://user-images.githubusercontent.com/93929892/196856651-c7a2596b-de87-472b-97a8-259e6c87ee12.png)

**3. Create a new Operator project.**

Execute the following command:
```CMD
mkdir sessionwebapp-operator  
cd sessionwebapp-operator  
operator-sdk init --plugins=ansible --domain hub.docker.com
```

output:
<img width="1317" alt="image" src="https://user-images.githubusercontent.com/93929892/204734366-9ae63f8d-95fa-46ce-a684-a40598b0b42d.png">


The initialization command will create the neccessary template and artifacts under the sessionwebapp-operator directory. 

Create a SessionWebAppOperator API.

Execute the following command:
```CMD
operator-sdk create api --group cache --version v1alpha1 --kind SessionWebAppOperator --generate-role
```

output:
<img width="1595" alt="image" src="https://user-images.githubusercontent.com/93929892/204734650-e49e3cf0-d198-4de0-9c19-2f15d02104a8.png">


Open the directory using any editor.

<img width="338" alt="image" src="https://user-images.githubusercontent.com/93929892/204735036-9ce0356a-af63-4e3a-93bf-0c7cbf342948.png">


**4. Reviewing the watches.yaml**

```yaml
- version: v1alpha1
  group: cache.hub.docker.com
  kind: SessionWebAppOperator
  role: sessionwebappoperator
```
  
The watches.yaml file connects the SessionWebAppOperator resource to the sessionwebappoperator ansible role.


**5. Update main.yaml**

Open the roles/sessionwebappoperator/tasks/main.yml file and update as follows.

```yaml
- name: start sessionwebappoperator app
  kubernetes.core.k8s:
    definition:
      kind: Deployment
      apiVersion: apps/v1
      metadata:
        name: webapp-deployment
        namespace: default
      spec:
        replicas: 1
        selector:
          matchLabels:
            app: webapp
        template:
          metadata:
            labels:
              app: webapp
          spec:
            containers:
            - name: webapp
              image: "docker.io/centraltechhub/sessionwebapp:v1"
              env:
                - name: ENV1
                  value: "{{env1}}"
              ports:
                - containerPort: 9080

- name: start sessionwebappoperator service
  kubernetes.core.k8s:
    definition:
      kind: Service
      apiVersion: v1
      metadata:
        name: webapp-deployment
        namespace: default
      spec:
        selector:
          app: webapp
        ports:
          - protocol: TCP
            port: 80
            targetPort: 9080
            nodePort: 30000
        type: LoadBalancer

```

**6. Edit the cache_v1alpha1_sessionwebappoperator.yaml**

```yaml
apiVersion: cache.hub.docker.com/v1alpha1
kind: SessionWebAppOperator
metadata:
  name: sessionwebappoperator-sample
spec:
  env1: "TestEnvironment1"
```

**7. Edit the Makefile**

```yaml
Image URL to use all building/pushing image targets
IMG ?= controller:latest
```
to 
```yaml
IMG ?= centraltechhub/operators:v1 # Replace this value with the relevant docker repository and version
```

**8. Edit the role.yaml file under config/rbac.**

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: manager-role
rules:
  ##
  ## Base operator rules
  ##
  - apiGroups:
      - ""
    resources:
      - secrets
      - pods
      - pods/exec
      - pods/log
      - services
    verbs:
      - create
      - delete
      - get
      - list
      - patch
      - update
      - watch
  - apiGroups:
      - apps
    resources:
      - deployments
      - daemonsets
      - replicasets
      - statefulsets
    verbs:
      - create
      - delete
      - get
      - list
      - patch
      - update
      - watch
```

**9.  Building the docker images and pushing it to the respository**

Ensure to save all the files. Execute the following command from the project directory.

```CMD
make docker-build docker-push
```

If the build is successful, you will be able to see the image present in the docker repository.

![image](https://user-images.githubusercontent.com/93929892/204740313-439a161c-346e-4fc3-b06a-9067efcdc545.png)

**10. Generate the operator install yaml file.** 

Execute the following command:

```CMD
kubectl kustomize config/default/ > config/default/sessionwebappoperator-install.yaml
```

The sessionwebappoperator-install.yaml will be generated in the sessionwebapp-operator/config/default directory. Open it and review.

Modify the following:
image: controller:latest to

```yaml
image: centraltechhub/operators:v1
```

save the file and exit.

**11. Install the operator.**

Navigate to the operator root directory and execute the following command:

```CMD
kubectl create -f config/default/sessionwebappoperator-install.yaml
```

output:
<img width="1284" alt="image" src="https://user-images.githubusercontent.com/93929892/204741972-fc70d2f6-e906-48d5-88de-ee8f8a47e0bd.png">


check if the operator is running.

<img width="1534" alt="image" src="https://user-images.githubusercontent.com/93929892/204742157-dacc8d3d-ae88-4f30-9ae8-b8e64bbd6f39.png">


**12. Install the sessionwebapp application container.**

Execute the following command:

```CMD
kubectl create -f config/samples/cache_v1alpha1_sessionwebappoperator.yaml
```
Check if both the pod and service is deployed sucessfully.

<img width="1558" alt="image" src="https://user-images.githubusercontent.com/93929892/204755444-5e7b4bff-2d14-44ba-9370-6a3cc6314f12.png">

Execute

```CMD
make deploy
```


**13. Start Minikube tunnel**

In a new command prompt, execute

```CMD
minikube tunnel
```
<img width="1333" alt="image" src="https://user-images.githubusercontent.com/93929892/204774482-c25f803f-92bf-49e9-9573-e22f0f48ed1c.png">


Post which external-ip will be assigned to the service.

<img width="1571" alt="image" src="https://user-images.githubusercontent.com/93929892/204756027-69792a88-0619-4105-9373-852b5eac6c13.png">

**14. Open browser and navigate http://localhost/sessionwebapp/ServerDetails.html**

![image](https://user-images.githubusercontent.com/93929892/204756370-e46d2b45-6072-4edf-9ce8-d482ec7faef7.png)

**15. Addtional testing.**

Login into the container using the following command (copy the right container name):

```CMD
kubectl exec -it webapp-deployment-d76bb8f96-c2hn8 bash
```
run the following command:
```CMD
echo $ENV1
```
The following will be printed: TestEnvironment1

Modify config/samples/cache_v1alpha1_sessionwebappoperator.yaml. Set env1: "TestEnvironment2".

Run the following:
```CMD
kubectl apply -f config/samples/cache_v1alpha1_sessionwebappoperator.yaml
```

Login into the container using the following command (copy the right container name):

```CMD
kubectl exec -it webapp-deployment-59c684c549-6c984 bash
```
run the following command:
```CMD
echo $ENV1
```
The following will be printed: TestEnvironment2

The End. Happy Learning !!

