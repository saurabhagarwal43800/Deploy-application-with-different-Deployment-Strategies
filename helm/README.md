# Configuring Helm:

Download [helm](https://drive.google.com/file/d/1RzmPqLKQHdl2hM2e5Qx8bo0DKaCCGyUF/view?usp=sharing) and [tiller](https://drive.google.com/file/d/17SBFnjH909bRay7jbKiftt9x2nmCoxU0/view?usp=sharing) program and set the path  

Run the following command to configure helm
```
$ helm init
$ helm repo add https://kubernetes_charts.storage.googleapis.com
$ helm repo list
$ helm repo update
$ kubectl -n kube-system create serviceaccount tiller
$ kubectl create clusterrolebinding tiller --cluster-role cluster-admin --serviceaccount=kube-system:tiller
$ helm init --service-account tiller --upgrade
```
