# **Description**

This repository contains the Citrix ADC CPX which is a container-based application delivery controller that can be provisioned on a Docker host. Learn more about Citrix ADC CPX [here](https://docs.citrix.com/en-us/citrix-adc-cpx/12-1/about.html).

# **Deployment Solutions**

There are two deployment solution present for Citrix ADC CPX in Google cloud.

* Stanalone CPX.
* Citrix ADC CPX with [Citrix Ingress Controller](https://github.com/citrix/citrix-k8s-ingress-controller) running in [side-car mode](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/).

# **Installation**
## **Command line instructions**
You can use [Google Cloud Shell](https://cloud.google.com/shell/) or a local
workstation to complete these steps.

[![Open in Cloud Shell](http://gstatic.com/cloudssh/images/open-btn.svg)](https://console.cloud.google.com/cloudshell/editor?cloudshell_git_repo=https://github.com/GoogleCloudPlatform/click-to-deploy&cloudshell_working_dir=k8s/jenkins)

### Prerequisites
#### Set up command-line tools
You'll need the following tools in your development environment. If you are using Cloud Shell, gcloud, kubectl, Docker, and Git are installed in your environment by default.

* [gcloud](https://cloud.google.com/sdk/gcloud/)
* [kubectl](https://kubernetes.io/docs/reference/kubectl/overview/)
* [docker](https://docs.docker.com/install/)
* [git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)
* [helm](https://helm.sh)

Configure `gcloud` as a Docker credential helper:

```shell
gcloud auth configure-docker
```

#### Create a Google Kubernetes Engine cluster
Create a cluster from the command line. If you already have a cluster that you
want to use, skip this step.
```shell
export CLUSTER=citrix-cpx
export ZONE=asia-south1-a
```
```shell
gcloud beta container --project <your-project-name> clusters create "$CLUSTER" --zone "$ZONE" --cluster-version "1.22.11-gke.400" --machine-type "n1-standard-2" --image-type "COS" --disk-type "pd-standard" --disk-size "100"  --num-nodes "3" --addons HorizontalPodAutoscaling,HttpLoadBalancing
```

#### Configure `kubectl` to connect to the cluster
Connect to the created Kubernetes cluster and create a cluster-admin role for your Google Account

```shell
gcloud container clusters get-credentials "$CLUSTER" --zone "$ZONE" --project <name-of-your-project>
```
Now your kubectl client is updated with the credentials required to login to the newly created Kubernetes cluster

```shell
kubectl create clusterrolebinding cpx-cluster-admin --clusterrole=cluster-admin --user=<email of the gcp account>
```

#### Clone this repo
Clone this repo. Go to citrix-adc-cpx-gcp-marketplace/ directory:
```shell
git clone https://github.com/citrix/citrix-adc-cpx-gcp-marketplace.git
cd citrix-adc-cpx-gcp-marketplace/
```

#### Install the Application resource definition
An Application resource is a collection of individual Kubernetes components,
such as Services, Deployments, and so on, that you can manage as a group.

To set up your cluster to understand Application resources, run the following
command:
```shell
make crd/install
```
You need to run this command once for each cluster.

The Application resource is defined by the
[Kubernetes SIG-apps](https://github.com/kubernetes/community/tree/master/sig-apps)
community. The source code can be found on
[github.com/kubernetes-sigs/application](https://github.com/kubernetes-sigs/application).

### **Install the Application**

The following table lists the configurable parameters of the CPX with Citrix ingress controller running as side-car chart and their default values.

| Parameters | Mandatory or Optional | Default value | Description |
| ---------- | --------------------- | ------------- | ----------- |
| license.accept | Mandatory | no | Set `yes` to accept the Citrix ingress controller end user license agreement. |
| replicaCount  | Optional  | 1 | Number of CPX-CIC pods to be deployed. |
| ADMSettings.licenseServerIP | Optional | N/A | Provide the Citrix Application Delivery Management (ADM) IP address to license Citrix ADC CPX. For more information, see [Licensing](https://developer-docs.citrix.com/projects/citrix-k8s-ingress-controller/en/latest/licensing/) |
| ADMSettings.licenseServerPort | Optional | 27000 | Citrix ADM port if non-default port is used. |
| ADMSettings.ADMIP | Optional | |  Citrix Application Delivery Management (ADM) IP address. |
| ADMSettings.ADMFingerPrint | Optional | N/A | Citrix Application Delivery Management (ADM) Finger Print. For more information, see [this](https://docs.citrix.com/en-us/citrix-application-delivery-management-service/application-analytics-and-management/service-graph.html). |
| ADMSettings.loginSecret | Optional | N/A | The secret key to login to the ADM. For information on how to create the secret keys, see [Prerequisites](#prerequistes). |
| ADMSettings.bandWidthLicense | Optional | False | Set to true if you want to use bandwidth based licensing for Citrix ADC CPX. |
| ADMSettings.bandWidth | Optional | N/A | Desired bandwidth capacity to be set for Citrix ADC CPX in Mbps. |
| ADMSettings.vCPULicense | Optional | N/A | Set to true if you want to use vCPU based licensing for Citrix ADC CPX. |
| ADMSettings.cpxCores | Optional | 1 | Desired number of vCPU to be set for Citrix ADC CPX. |
| cic.required | Mandatory | true | CIC to be run as sidecar with Citrix ADC CPX |
| nsProtocol | Optional | http | Protocol http or https used for the communication between Citrix Ingress Controller and CPX |
| logLevel | Optional | DEBUG | The loglevel to control the logs generated by CIC. The supported loglevels are: CRITICAL, ERROR, WARNING, INFO, DEBUG and TRACE. For more information, see [Logging](https://github.com/citrix/citrix-k8s-ingress-controller/blob/master/docs/configure/log-levels.md).|
| jsonLog | Optional | false | Set this argument to true if log messages are required in JSON format |
| entityPrefix | Optional | k8s | The prefix for the resources on the Citrix ADC CPX. |
| nitroReadTimeout | Optional | 20 | The nitro Read timeout in seconds, defaults to 20 |
| kubernetesURL | Optional | N/A | The kube-apiserver url that CIC uses to register the events. If the value is not specified, CIC uses the internal kube-apiserver IP address. |
| disableAPIServerCertVerify | Optional | False | Set this parameter to True for disabling API Server certificate verification. |
| ingressClass | Optional | N/A | If multiple ingress load balancers are used to load balance different ingress resources. You can use this parameter to specify Citrix ingress controller to configure Citrix ADC associated with specific ingress class. For more information on Ingress class, see [Ingress class support](https://developer-docs.citrix.com/projects/citrix-k8s-ingress-controller/en/latest/configure/ingress-classes/). For Kubernetes version >= 1.19, this will create an IngressClass object with the name specified here |
| setAsDefaultIngressClass | Optional | False | Set the IngressClass object as default. New Ingresses without an "ingressClassName" field specified will be assigned the class specified in ingressClass. Applicable only for kubernetes versions >= 1.19 |
| updateIngressStatus | Optional | False | Set this argument if you want to update ingress status of the ingress resources exposed via CPX. This is only applicable if servicetype of CPX service is LoadBalancer. |
| defaultSSLCertSecret | Optional | N/A | Provide Kubernetes secret name that needs to be used as a default non-SNI certificate in Citrix ADC. |
| nsHTTP2ServerSide | Optional | OFF | Set this argument to `ON` for enabling HTTP2 for Citrix ADC service group configurations. |
| nsCookieVersion | Optional | 0 | Specify the persistence cookie version (0 or 1). |
| nsConfigDnsRec | Optional | false | To enable/disable DNS address Record addition in ADC through Ingress |
| nsSvcLbDnsRec | Optional | false | To enable/disable DNS address Record addition in ADC through Type Load Balancer Service |
| nsDnsNameserver | Optional | N/A | To add DNS Nameservers in ADC |
| nsLbHashAlgo.required | Optional | false | Set this value to set the LB consistent hashing Algorithm |
| nsLbHashAlgo.hashFingers | Optional |256 | Specifies the number of fingers to be used for hashing algorithm. Possible values are from 1 to 1024, Default value is 256 |
| nsLbHashAlgo.hashAlgorithm | Optional | 'default' | Specifies the supported algorithm. Supported algorithms are "default", "jarh", "prac", Default value is 'default' |
| ipam | Optional | False | Set this argument if you want to use the IPAM controller to automatically allocate an IP address to the service of type LoadBalancer. |
| serviceType.loadBalancer.enabled | Optional | False | Set this argument if you want servicetype of CPX service to be LoadBalancer. |
| serviceType.nodePort.enabled | Optional | False | Set this argument if you want servicetype of CPX service to be NodePort. |
| serviceType.nodePort.httpPort | Optional | N/A | Specify the HTTP nodeport to be used for NodePort CPX service. |
| serviceType.nodePort.httpsPort | Optional | N/A | Specify the HTTPS nodeport to be used for NodePort CPX service. |
| serviceSpec.externalTrafficPolicy | Optional | Cluster | Use this parameter to provide externalTrafficPolicy for CPX service of type LoadBalancer or NodePort. `serviceType.loadBalancer.enabled` or `serviceType.nodePort.enabled` should be set to `true` according to your use case for using this parameter. |
| serviceSpec.loadBalancerIP | Optional | N/A | Use this parameter to provide LoadBalancer IP to CPX service of type LoadBalancer. `serviceType.loadBalancer.enabled` should be set to `true` for using this parameter. |
| exporter.required | Optional | false | Use the argument if you want to run the [Exporter for Citrix ADC Stats](https://github.com/citrix/citrix-adc-metrics-exporiter) along with Citrix ingress controller to pull metrics for the Citrix ADC CPX|
| exporter.ports.containerPort | Optional | 8888 | The Exporter for Citrix ADC Stats container port. |
| crds.install | Optional | False | Unset this argument if you don't want to install CustomResourceDefinitions which are consumed by CIC. |
| crds.retainOnDelete | Optional | false | Set this argument if you want to retain CustomResourceDefinitions even after uninstalling CIC. This will avoid data-loss of Custom Resource Objects created before uninstallation. |
| analyticsConfig.required | Mandatory | false | Set this to true if you want to configure Citrix ADC to send metrics and transaction records to COE. |
| analyticsConfig.distributedTracing.enable | Optional | false | Set this value to true to enable OpenTracing in Citrix ADC. |
| analyticsConfig.distributedTracing.samplingrate | Optional | 100 | Specifies the OpenTracing sampling rate in percentage. |
| analyticsConfig.endpoint.server | Optional | N/A | Set this value as the IP address or DNS address of the  analytics server. |
| analyticsConfig.endpoint.service | Optional | N/A | Set this value as the IP address or service name with namespace of the analytics service deployed in Kubernetes. Format: namespace/servicename. |
| analyticsConfig.timeseries.port | Optional | 5563 | Specify the port used to expose COE service for timeseries endpoint. |
| analyticsConfig.timeseries.metrics.enable | Optional | Set this value to true to enable sending metrics from Citrix ADC. |
| analyticsConfig.timeseries.metrics.mode | Optional | avro |  Specifies the mode of metric endpoint. |
| analyticsConfig.timeseries.auditlogs.enable | Optional | false | Set this value to true to export audit log data from Citrix ADC. |
| analyticsConfig.timeseries.events.enable | Optional | false | Set this value to true to export events from the Citrix ADC. |
| analyticsConfig.transactions.enable | Optional | false | Set this value to true to export transactions from Citrix ADC. |
| analyticsConfig.transactions.port | Optional | 5557 | Specify the port used to expose COE service for transaction endpoint. |
| nodeSelector.key | Optional | N/A | Node label key to be used for nodeSelector option for CPX-CIC deployment. |
| nodeSelector.value | Optional | N/A | Node label value to be used for nodeSelector option in CPX-CIC deployment. |


Assign values to the required parameters:
```shell
CITRIX_NAME=citrix-1
CITRIX_NAMESPACE=default
CITRIX_SERVICEACCOUNT=cic-k8s-role
```

Create a service account with required permissions:
```shell
cat service_account.yaml | sed -e "s/{NAMESPACE}/$CITRIX_NAMESPACE/g" -e "s/{SERVICEACCOUNTNAME}/$CITRIX_SERVICEACCOUNT/g" | kubectl create -f -
```

> NOTE: The above are the mandatory parameters. In addition to these you can also assign values to the parameters mentioned in the above table.

Create a template for the chart using the parameters you want to set:
```
helm template $CITRIX_NAME chart/citrix-cpx-with-ingress-controller \
  --namespace $CITRIX_NAMESPACE \
  --set license.accept=yes \
  --set serviceAccount=$CITRIX_SERVICEACCOUNT > /tmp/$CITRIX_NAME.yaml
```

Finally, deploy the chart:
```shell
kubectl create -f /tmp/$CITRIX_NAME.yaml -n $CITRIX_NAMESPACE
```

### **Uninstall the Application**
Delete the application, service account and cluster:
```shell
kubectl delete -f /tmp/$CITRIX_NAME.yaml -n $CITRIX_NAMESPACE
cat service_account.yaml | sed -e "s/{NAMESPACE}/$CITRIX_NAMESPACE/g" -e "s/{SERVICEACCOUNTNAME}/$CITRIX_SERVICEACCOUNT/g" | kubectl delete -f -
gcloud container clusters delete "$CLUSTER" --zone "$ZONE"
```

# **Code of Conduct**
This project adheres to the [Kubernetes Community Code of Conduct](https://github.com/kubernetes/community/blob/master/code-of-conduct.md). By participating in this project you agree to abide by its terms.

