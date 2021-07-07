To install minikube

MacOS: brew install minikube

To create AWX Operator minikube

This Kubernetes Operator is meant to be deployed in your Kubernetes cluster(s) and can manage one or more AWX instances in any namespace.

For testing purposes, the awx-operator can be deployed on a Minikube cluster.

    $ minikube start --addons=ingress --cpus=4 --cni=flannel --install-addons=true \
       --kubernetes-version=stable --memory=6g --wait=false

    😄  minikube v1.21.0 on Darwin 11.4
    ✨  Using the virtualbox driver based on user configuration
    👍  Starting control plane node minikube in cluster minikube
    🔥  Creating virtualbox VM (CPUs=4, Memory=6144MB, Disk=20000MB) ...
    🐳  Preparing Kubernetes v1.20.7 on Docker 20.10.6 ...
    ▪ Generating certificates and keys ...
    ▪ Booting up control plane ...
    ▪ Configuring RBAC rules ...
    🔗  Configuring Flannel (Container Networking Interface) ...
    🔎  Verifying Kubernetes components...
    ▪ Using image docker.io/jettech/kube-webhook-certgen:v1.5.1
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
    ▪ Using image docker.io/jettech/kube-webhook-certgen:v1.5.1
    ▪ Using image k8s.gcr.io/ingress-nginx/controller:v0.44.0
    🔎  Verifying ingress addon...
    🌟  Enabled addons: storage-provisioner, default-storageclass, ingress
    🏄  Done! kubectl is now configured to use "minikube" cluster and "default"   namespace by default

Once minikube is deployed, check if the node(s) and kube-apiserver is up and working

    $ kubectl get nodes
    NAME       STATUS   ROLES                  AGE    VERSION
    minikube   Ready    control-plane,master   111s   v1.20.7

    $ kubectl get pods -A
    NAMESPACE       NAME                                        READY   STATUS          RESTARTS   AGE
    ingress-nginx   ingress-nginx-admission-create-vkwf5        0/1     Completed       0          97s
    ingress-nginx   ingress-nginx-admission-patch-vrmff         0/1     Completed       0          97s
    ingress-nginx   ingress-nginx-controller-5d88495688-sq8kx   1/1     Running         0          97s
    kube-system     coredns-74ff55c5b-v5czz                     1/1     Running         0          97s
    kube-system     etcd-minikube                               1/1     Running         0          112s
    kube-system     kube-apiserver-minikube                     1/1     Running         0          112s
    kube-system     kube-controller-manager-minikube            1/1     Running         0          112s
    kube-system     kube-flannel-ds-amd64-nrt4w                 1/1     Running         0          97s
    kube-system     kube-proxy-vmhwn                            1/1     Running         0          97s
    kube-system     kube-scheduler-minikube                     1/1     Running         0          112s
    kube-system     storage-provisioner                         1/1     Running         1          112s

Deploy AWX Operator into cluster:

    $ kubectl apply -f https://raw.githubusercontent.com/ansible/awx-operator/0.12.0/deploy/awx-operator.yaml
    customresourcedefinition.apiextensions.k8s.io/awxs.awx.ansible.com created
    customresourcedefinition.apiextensions.k8s.io/awxbackups.awx.ansible.com created
    customresourcedefinition.apiextensions.k8s.io/awxrestores.awx.ansible.com created
    clusterrole.rbac.authorization.k8s.io/awx-operator created
    clusterrolebinding.rbac.authorization.k8s.io/awx-operator created
    serviceaccount/awx-operator created
    deployment.apps/awx-operator created

Wait a few minutes and you should have the awx-operator running.

    $ kubectl get pods
    NAME                           READY   STATUS    RESTARTS   AGE
    awx-operator-79bc95f78-q2lnq   1/1     Running   0          46s

Then create a file named awx-demo.yml with the suggested content. The metadata.name you provide, will be the name of the resulting AWX deployment.
If you deploy more than one AWX instance to the same namespace, be sure to use unique names.


    ---
    apiVersion: awx.ansible.com/v1beta1
    kind: AWX
    metadata:
      name: awx-demo
    spec:
      service_type: nodeport
      ingress_type: none
      hostname: awx-demo.example.com

Finally, use kubectl to create the awx instance in your cluster:

    $ kubectl apply -f awx-demo.yml
    awx.awx.ansible.com/awx-demo created

After a few minutes, the new AWX instance will be deployed. One can look at the operator pod logs in order to know where the installation process is at.
This can be done by running the following command:

    $ kubectl logs -f deployments/awx-operator

    /**** Output should end like this ****/

    --------------------------- Ansible Task Status Event StdOut  -----------------

    PLAY RECAP *********************************************************************
    localhost                  : ok=53   changed=0    unreachable=0    failed=0        skipped=39   rescued=0    ignored=0


    $ kubectl get pods -l "app.kubernetes.io/managed-by=awx-operator"
    NAME                       READY   STATUS    RESTARTS   AGE
    awx-demo-9975db9b6-mx8fr   4/4     Running   0          10m
    awx-demo-postgres-0        1/1     Running   0          10m

    $ kubectl get svc -l "app.kubernetes.io/managed-by=awx-operator"
    NAME                TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
    awx-demo-postgres   ClusterIP   None           <none>        5432/TCP       10m
    awx-demo-service    NodePort    10.97.67.242   <none>        80:30997/TCP   10m

Once deployed, the AWX instance will be accessible by the command

    $ minikube service awx-demo-service --url
    http://192.168.99.101:30997

Get the Admin Password for Ansible AWX

By default, the admin user is admin and the password is available in the <resourcename>-admin-password secret.

    $ kubectl get secret awx-demo-admin-password \
      -o jsonpath="{.data.password}" | base64 --decode
    lxQ8uWlE9Wevkgmy5Kx2AqFdY80v34gx
