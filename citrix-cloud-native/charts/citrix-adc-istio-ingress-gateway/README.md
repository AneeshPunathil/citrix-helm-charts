# Deploy Citrix ADC as an Ingress Gateway in Istio environment using Helm charts

Citrix Application Delivery Controller (ADC) can be deployed as an Istio Ingress Gateway to control the ingress traffic to Istio service mesh.

# Table of Contents
1. [TL; DR;](#tldr)
2. [Introduction](#introduction)
3. [Deploy Citrix ADC VPX or MPX as an Ingress Gateway](#deploy-citrix-adc-vpx-or-mpx-as-an-ingress-gateway)
4. [Deploy Citrix ADC CPX as an Ingress Gateway](#deploy-citrix-adc-cpx-as-an-ingress-gateway)
5. [Using Existing Certificates to deploy Citrix ADC as an Ingress Gateway](#using-existing-certificates-to-deploy-citrix-adc-as-an-ingress-gateway)
6. [Deploy Citrix ADC as an Ingress Gateway in multi cluster Istio Service mesh](#deploy-citrix-adc-as-a-multicluster-ingress-gateway)
7. [Segregating traffic with multiple Ingress Gateways](#segregating-traffic-with-multiple-ingress-gateways)
8. [Visualizing statistics of Citrix ADC Ingress Gateway with Metrics Exporter](#visualizing-statistics-of-citrix-adc-ingress-gateway-with-metrics-exporter)
9. [Exposing services running on non-HTTP ports](#exposing-services-running-on-non-http-ports)
10. [Generate Certificate for Ingress Gateway](#generate-certificate-for-ingress-gateway)
11. [Citrix ADC CPX License Provisioning](#citrix-adc-cpx-license-provisioning)
12. [Citrix ADC as Ingress Gateway: a sample deployment](#citrix-adc-as-ingress-gateway-a-sample-deployment)
13. [Uninstalling the Helm chart](#uninstalling-the-helm-chart)
14. [Citrix ADC VPX/MPX Certificate Verification](#citrix-adc-vpx-or-mpx-certificate-verification)
15. [Configuration Parameters](#configuration-parameters)


## <a name="tldr">TL; DR;</a>

### To deploy Citrix ADC VPX or MPX as an Ingress Gateway:

       kubectl create secret generic nslogin --from-literal=username=<citrix-adc-user> --from-literal=password=<citrix-adc-password> -n citrix-system

       helm repo add citrix https://citrix.github.io/citrix-helm-charts/

       helm install citrix-adc-istio-ingress-gateway citrix/citrix-cloud-native --namespace citrix-system --set iaIngress.enabled=true,iaIngress.ingressGateway.EULA=YES,iaIngress.ingressGateway.netscalerUrl=https://<nsip>[:port],iaIngress.ingressGateway.vserverIP=<IPv4 Address>,iaIngress.ingressGateway.adcServerName=<ADC Cert Server Name> --set iaIngress.secretName=nslogin

### To deploy Citrix ADC CPX as an Ingress Gateway:

       helm repo add citrix https://citrix.github.io/citrix-helm-charts/

       helm install citrix-adc-istio-ingress-gateway citrix/citrix-cloud-native --namespace citrix-system --set iaIngress.enabled=true,iaIngress.ingressGateway.EULA=YES --set iaIngress.citrixCPX=true


## <a name="introduction">Introduction</a>

This chart deploys Citrix ADC VPX, MPX, or CPX as an Ingress Gateway in the Istio service mesh using the Helm package manager. For detailed information on different deployment options, see [Deployment Architecture](https://github.com/citrix/citrix-istio-adaptor/blob/master/docs/istio-integration/architecture.md).

### Prerequisites

The following prerequisites are required for deploying Citrix ADC as an Ingress Gateway in Istio service mesh:

- Ensure that **Istio version 1.6.4 onwards** is installed
- Ensure that Helm with version 3.x is installed. Follow this [step](https://github.com/citrix/citrix-helm-charts/blob/master/Helm_Installation_version_3.md) to install the same.
- Ensure that your cluster has Kubernetes version 1.14.0 or later and the `admissionregistration.k8s.io/v1beta1` API is enabled
- **For deploying Citrix ADC VPX or MPX as an Ingress gateway:**

  Create a Kubernetes secret for the Citrix ADC user name and password using the following command:
  
        kubectl create secret generic nslogin --from-literal=username=<citrix-adc-user> --from-literal=password=<citrix-adc-password> -n citrix-system

You can verify the API by using the following command:

        kubectl api-versions | grep admissionregistration.k8s.io/v1beta1

The following output indicates that the API is enabled:

        admissionregistration.k8s.io/v1beta1

- **Registration of Citrix ADC CPX in ADM**

Create a secret for ADM username and password

        kubectl create secret generic admlogin --from-literal=username=<adm-username> --from-literal=password=<adm-password> -n citrix-system

- **Important Note:** For deploying Citrix ADC VPX or MPX as ingress gateway, you should establish the connectivity between Citrix ADC VPX or MPX and cluster nodes. This connectivity can be established by configuring routes on Citrix ADC as mentioned [here](https://github.com/citrix/citrix-k8s-ingress-controller/blob/master/docs/network/staticrouting.md) or by deploying [Citrix Node Controller](https://github.com/citrix/citrix-k8s-node-controller).
  

## <a name="deploy-citrix-adc-vpx-or-mpx-as-an-ingress-gateway">Deploy Citrix ADC VPX or MPX as an Ingress Gateway</a>

 To deploy Citrix ADC VPX or MPX as an Ingress Gateway in the Istio service mesh, do the following step. In this example, release name is specified as `citrix-adc-istio-ingress-gateway` and namespace as `citrix-system`.

        kubectl create secret generic nslogin --from-literal=username=<citrix-adc-user> --from-literal=password=<citrix-adc-password> -n citrix-system
        
        helm repo add citrix https://citrix.github.io/citrix-helm-charts/

        helm install citrix-adc-istio-ingress-gateway citrix/citrix-cloud-native --namespace citrix-system --set iaIngress.enabled=true,iaIngress.ingressGateway.EULA=YES,iaIngress.secretName=nslogin,iaIngress.ingressGateway.netscalerUrl=https://<nsip>[:port],iaIngress.ingressGateway.vserverIP=<IPv4 Address>


## <a name="deploy-citrix-adc-cpx-as-an-ingress-gateway">Deploy Citrix ADC CPX as an Ingress Gateway</a>

 To deploy Citrix ADC CPX as an Ingress Gateway, do the following step. In this example, release name is specified as `my-release` and namespace is used as `citrix-system`.

        helm repo add citrix https://citrix.github.io/citrix-helm-charts/

        helm install citrix-adc-istio-ingress-gateway citrix/citrix-cloud-native --namespace citrix-system --set iaIngress.enabled=true,iaIngress.ingressGateway.EULA=YES,iaIngress.citrixCPX=true


## <a name="deploy-citrix-adc-as-a-multicluster-ingress-gateway">Deploy Citrix ADC as an Ingress Gateway in multi cluster Istio Service mesh</a>

To deploy **Citrix ADC VPX/MPX as an Ingress Gateway** in multi cluster Istio Service mesh, carry out below steps.
```
helm repo add citrix https://citrix.github.io/citrix-helm-charts/

helm install citrix-adc-istio-ingress-gateway citrix/citrix-cloud-native --namespace citrix-system --set iaIngress.enabled=true,iaIngress.ingressGateway.EULA=YES,iaIngress.ingressGateway.netscalerUrl=https://<nsip>[:port],iaIngress.ingressGateway.vserverIP=<IPv4 Address> --set iaIngress.ingressGateway.multiClusterIngress=true 
```

To deploy **Citrix ADC CPX as an Ingress Gateway** in multi cluster Istio Service mesh, carry out below steps.
```
helm repo add citrix https://citrix.github.io/citrix-helm-charts/

helm install citrix-adc-istio-ingress-gateway citrix/citrix-cloud-native --namespace citrix-system --set iaIngress.enabled=true,iaIngress.ingressGateway.EULA=YES --set iaIngress.citrixCPX=true --set iaIngress.ingressGateway.multiClusterIngress=true 

```
By default, port 15443 of the Citrix ADC will be used to handle all the inter-cluster traffic coming to services deployed in local cluster. These services are exposed using `*.global` domain.
To modify the default 15443 port and "global" domain, use _ingressGateway.multiClusterListenerPort_ and _ingressGateway.multiClusterSvcDomain_ options of helm chart.

For example, to use port 25443 and _mydomain_ as the service domain to expose local cluster deployed services to services in remote clusters.

```

helm install citrix-adc-istio-ingress-gateway citrix/citrix-cloud-native --namespace citrix-system --set iaIngress.enabled=true,iaIngress.ingressGateway.EULA=YES,iaIngress.ingressGateway.netscalerUrl=https://<nsip>[:port],iaIngress.ingressGateway.vserverIP=<IPv4 Address> --set iaIngress.ingressGateway.multiClusterIngress=true --set iaIngress.ingressGateway.multiClusterListenerPort=25443 --set iaIngress.ingressGateway.multiClusterSvcDomain=mydomain

```
Follow [this](https://github.com/citrix/citrix-helm-charts/tree/master/examples/citrix-adc-ingress-in-multicluster-istio/README.md) as a sample example to deploy Citrix ADC as Ingress gateway in multi-cluster Istio service mesh.


## <a name="using-existing-certificates-to-deploy-citrix-adc-as-an-ingress-gateway">Using Existing Certificates to deploy Citrix ADC as an Ingress Gateway</a>

You may want to use the existing certificate and key for authenticating access to an application using Citrix ADC Ingress Gateway. In that case, you can create a Kubernetes secret from the existing certificate and key. You can mount the Kubernetes secret as data volumes in Citrix ADC Ingress Gateway.

To create a Kubernetes secret using an existing key named `test_key.pem` and a certificate named `test.pem`, use the following command:

        kubectl create -n citrix-system secret tls citrix-ingressgateway-certs --key test_key.pem --cert test.pem 

Note: Ensure that Kubernetes secret is created in the same namespace where Citrix ADC Ingress Gateway is deployed.

To deploy Citrix ADC VPX or MPX with secret volume, do the following step:

        kubectl create secret generic nslogin --from-literal=username=<citrix-adc-user> --from-literal=password=<citrix-adc-password> -n citrix-system

        helm repo add citrix https://citrix.github.io/citrix-helm-charts/

        helm install citrix-adc-istio-ingress-gateway citrix/citrix-cloud-native --namespace citrix-system --set iaIngress.enabled=true,iaIngress.ingressGateway.EULA=YES,iaIngress.secretName=nslogin,iaIngress.ingressGateway.netscalerUrl=https://<nsip>[:port],iaIngress.ingressGateway.vserverIP=<IPv4 Address>,iaIngress.ingressGateway.secretVolumes[0].name=test-ingressgateway-certs,iaIngress.ingressGateway.secretVolumes[0].secretName=test-ingressgateway-certs,iaIngress.ingressGateway.secretVolumes[0].mountPath=/etc/istio/test-ingressgateway-certs


To deploy Citrix ADC CPX with secret volume, do the following step:

        helm repo add citrix https://citrix.github.io/citrix-helm-charts/

        helm install citrix-adc-istio-ingress-gateway citrix/citrix-cloud-native --namespace citrix-system --set iaIngress.enabled=true,iaIngress.ingressGateway.EULA=YES,iaIngress.citrixCPX=true,iaIngress.ingressGateway.secretVolumes[0].name=test-ingressgateway-certs,iaIngress.ingressGateway.secretVolumes[0].secretName=test-ingressgateway-certs,iaIngress.ingressGateway.secretVolumes[0].mountPath=/etc/istio/test-ingressgateway-certs


## <a name="segregating-traffic-with-multiple-ingress-gateways">Segregating traffic with multiple Ingress Gateways</a>

You can deploy multiple Citrix ADC Ingress Gateway devices and segregate traffic to various deployments in the Istio service mesh. This can be achieved with *custom labels*. By default, Citrix ADC Ingress Gateway service comes up with the `app: citrix-ingressgateway` label. This label is used as a selector while deploying the Ingress Gateway or virtual service resources. If you want to deploy Ingress Gateway with the custom label, you can do it using the `iaIngress.ingressGateway.label` option in the Helm chart. 

To deploy Citrix ADC CPX Ingress Gateway with the label `my_custom_ingressgateway`, do the following step:
        
        helm repo add citrix https://citrix.github.io/citrix-helm-charts/

        helm install citrix-adc-istio-ingress-gateway citrix/citrix-cloud-native --namespace citrix-system --set iaIngress.enabled=true,iaIngress.ingressGateway.EULA=YES,iaIngress.citrixCPX=true,iaIngress.ingressGateway.lightWeightCPX=NO,iaIngress.ingressGateway.label=my_custom_ingressgateway


To deploy Citrix ADC VPX or MPX as an Ingress Gateway with the label `my_custom_ingressgateway`, do the following step:
        
        kubectl create secret generic nslogin --from-literal=username=<citrix-adc-user> --from-literal=password=<citrix-adc-password> -n citrix-system

        helm repo add citrix https://citrix.github.io/citrix-helm-charts/

        helm install citrix-adc-istio-ingress-gateway citrix/citrix-cloud-native --namespace citrix-system --set iaIngress.enabled=true,iaIngress.ingressGateway.EULA=YES,iaIngress.secretName=nslogin,iaIngress.ingressGateway.netscalerUrl=https://<nsip>[:port],iaIngress.ingressGateway.vserverIP=<IPv4 Address>,iaIngress.ingressGateway.label=my_custom_ingressgateway


## <a name="visualizing-statistics-of-citrix-adc-ingress-gateway-with-metrics-exporter">Visualizing statistics of Citrix ADC Ingress Gateway with Metrics Exporter</a>

By default, [Citrix ADC Metrics Exporter](https://github.com/citrix/citrix-adc-metrics-exporter) is also deployed along with Citrix ADC Ingress Gateway. Citrix ADC Metrics Exporter fetches statistical data from Citrix ADC and exports it to Prometheus running in Istio service mesh. When you add Prometheus as a data source in Grafana, you can visualize this statistical data in the Grafana dashboard.

Metrics Exporter requires the IP address of Citrix ADC CPX or VPX Ingress Gateway. It is retrieved from the value specified for `iaIngress.ingressGateway.netscalerUrl`.

When Citrix ADC CPX is deployed as Ingress Gateway, Metrics Exporter runs along with Citrix CPX Ingress Gateway in the same pod and specifying IP address is optional.

To deploy Citrix ADC as Ingress Gateway without Metrics Exporter, set the value of `iaIngress.metricExporter.required` as false.


        kubectl create secret generic nslogin --from-literal=username=<citrix-adc-user> --from-literal=password=<citrix-adc-password> -n citrix-system
    
        helm repo add citrix https://citrix.github.io/citrix-helm-charts/

        helm install citrix-adc-istio-ingress-gateway citrix/citrix-cloud-native --namespace citrix-system --set iaIngress.enabled=true,iaIngress.ingressGateway.EULA=YES,iaIngress.secretName=nslogin,iaIngress.ingressGateway.netscalerUrl=https://<nsip>[:port],iaIngress.ingressGateway.vserverIP=<IPv4 Address>,iaIngress.metricExporter.required=false


"Note:" To remotely access telemetry addons such as Prometheus and Grafana, see [Remotely Accessing Telemetry Addons](https://istio.io/docs/tasks/telemetry/gateways/).

## <a name="exposing-services-running-on-non-http-ports">Exposing services running on non-HTTP ports</a>

By default, services running on HTTP ports (80 & 443) are exposed through Citrix ADC Ingress Gateway. Similarly, you can expose services that are deployed on non-HTTP ports through the Citrix ADC Ingress Gateway device.

To deploy Citrix ADC MPX or VPX, and expose a service running on a TCP port, do the following step.

In this example, a service running on TCP port 5000 is exposed using port 10000 on Citrix ADC.

        kubectl create secret generic nslogin --from-literal=username=<citrix-adc-user> --from-literal=password=<citrix-adc-password> -n citrix-system

        helm repo add citrix https://citrix.github.io/citrix-helm-charts/

        helm install citrix-adc-istio-ingress-gateway citrix/citrix-cloud-native --namespace citrix-system --set iaIngress.enabled=true,iaIngress.ingressGateway.EULA=YES,iaIngress.secretName=nslogin,iaIngress.ingressGateway.netscalerUrl=https://<nsip>[:port],iaIngress.ingressGateway.vserverIP=<IPv4 Address>,iaIngress.ingressGateway.tcpPort[0].name=tcp1,iaIngress.ingressGateway.tcpPort[0].port=10000,iaIngress.ingressGateway.tcpPort[0].targetPort=5000

 To deploy Citrix ADC CPX and expose a service running on a TCP port, do the following step.
 In this example, port 10000 on the Citrix ADC CPX instance is exposed using TCP port 30000 (node port configuration) on the host machine.

        helm repo add citrix https://citrix.github.io/citrix-helm-charts/

        helm install citrix-adc-istio-ingress-gateway citrix/citrix-cloud-native --namespace citrix-system --set iaIngress.enabled=true,iaIngress.ingressGateway.EULA=YES,iaIngress.citrixCPX=true,iaIngress.ingressGateway.tcpPort[0].name=tcp1,iaIngress.ingressGateway.tcpPort[0].nodePort=30000,iaIngress.ingressGateway.tcpPort[0].port=10000,iaIngress.ingressGateway.tcpPort[0].targetPort=5000

## <a name="generate-certificate-for-ingress-gateway">Generate Certificate for Ingress Gateway </a>

Citrix Ingress gateway needs TLS certificate-key pair for establishing secure communication channel with backend applications. Earlier these certificates were issued by Istio Citadel and bundled in Kubernetes secret. Certificate was loaded in the application pod by doing volume mount of secret. Now `xDS-Adaptor` can generate its own certificate and get it signed by the Istio Citadel (Istiod). This eliminates the need of secret and associated [risks](https://kubernetes.io/docs/concepts/configuration/secret/#risks). 

xDS-Adaptor needs to be provided with details Certificate Authority (CA) for successful signing of Certificate Signing Request (CSR). By default, CA is `istiod.istio-system.svc` which accepts CSRs on port 15012. 
To skip this process, don't provide any value (empty string) to `iaIngress.certProvider.caAddr`.
```
        helm repo add citrix https://citrix.github.io/citrix-helm-charts/

        helm install citrix-adc-istio-ingress-gateway citrix/citrix-cloud-native --namespace citrix-system --set iaIngress.enabled=true,iaIngress.ingres    sGateway.EULA=YES,iaIngress.citrixCPX=true --set iaIngress.certProvider.caAddr=""
```

### <a name="using-third-party-service-account-tokens">Configure Third Party Service Account Tokens</a>

In order to generate certificate for application workload, xDS-Adaptor needs to send valid service account token along with Certificate Signing Request (CSR) to the Istio control plane (Citadel CA). Istio control plane authenticates the xDS-Adaptor using this JWT. 
Kubernetes supports two forms of these tokens:

* Third party tokens, which have a scoped audience and expiration.
* First party tokens, which have no expiration and are mounted into all pods.
 
 If Kubernetes cluster is installed with third party tokens, then the same information needs to be provided for automatic sidecar injection by passing `--set iaIngress.certProvider.jwtPolicy="third-party-jwt"`. By default, it is `first-party-jwt`.

```
        helm repo add citrix https://citrix.github.io/citrix-helm-charts/

        helm install cpx-sidecar-injector citrix/citrix-cpx-istio-sidecar-injector --namespace citrix-system --set iaIngress.cpxProxy.EULA=YES --set iaIngress.certProvider.caAddr="istiod.istio-system.svc" --set iaIngress.certProvider.jwtPolicy="third-party-jwt"

```

To determine if your cluster supports third party tokens, look for the TokenRequest API using below command. If there is no output, then it is `first-party-jwt`. In case of `third-party-jwt`, output will be like below.

```
# kubectl get --raw /api/v1 | jq '.resources[] | select(.name | index("serviceaccounts/token"))'

{
    "name": "serviceaccounts/token",
    "singularName": "",
    "namespaced": true,
    "group": "authentication.k8s.io",
    "version": "v1",
    "kind": "TokenRequest",
    "verbs": [
        "create"
    ]
}

```


## <a name="citrix-adc-cpx-license-provisioning">**Citrix ADC CPX License Provisioning**</a>
By default, CPX runs with 20 Mbps bandwidth called as [CPX Express](https://www.citrix.com/en-in/products/citrix-adc/cpx-express.html) however for better performance and production deployment customer needs licensed CPX instances. [Citrix ADM](https://www.citrix.com/en-in/products/citrix-application-delivery-management/) is used to check out licenses for Citrix ADC CPX.

**Bandwidth based licensing**
For provisioning licensing on Citrix ADC CPX, it is mandatory to provide License Server information to CPX. This can be done by setting **ADMSettings.licenseServerIP** as License Server IP. In addition to this, **ADMSettings.bandWidthLicense** needs to be set true and desired bandwidth capacity in Mbps should be set **ADMSettings.bandWidth**.
For example, to set 2Gbps as bandwidth capacity, below command can be used.

	helm repo add citrix https://citrix.github.io/citrix-helm-charts/

        helm install citrix-adc-istio-ingress-gateway citrix/citrix-adc-istio-ingress-gateway --namespace citrix-system --set iaIngress.enabled=true,iaIngress.ingressGateway.EULA=YES --set iaIngress.ADMSettings.licenseServerIP=<Licenseserver_IP>,iaIngress.ADMSettings.bandWidthLicense=True --set iaIngress.ADMSettings.bandWidth=2000 --set iaIngress.citrixCPX=true

## <a name="citrix-adc-as-ingress-gateway-a-sample-deployment">Citrix ADC as Ingress Gateway: a sample deployment</a>

A sample deployment of Citrix ADC as an Ingress gateway for the Bookinfo application is provided [here](https://github.com/citrix/citrix-helm-charts/tree/master/examples/citrix-adc-in-istio).

## <a name="uninstalling-the-helm-chart">Uninstalling the Helm chart</a>

To uninstall or delete a chart with release name as `citrix-adc-istio-ingress-gateway`, do the following step.

        helm delete citrix-adc-istio-ingress-gateway

The command removes all the Kubernetes components associated with the chart and deletes the release.

## <a name="citrix-adc-vpx-or-mpx-certificate-verification">Citrix ADC VPX/MPX Certificate Verification</a>

Create a Kubernetes secret holding the CA certificate of Citrix ADC VPX/MPX with the filename `root-cert.pem`.

        kubectl create secret generic citrix-adc-cert --from-file=./root-cert.pem

Note: Ensure that Kubernetes secret is created in the same namespace where Citrix ADC Ingress Gateway is deployed.

To deploy Citrix ADC VPX or MPX with Citrix ADC certificate verification, do the following step:

        kubectl create secret generic nslogin --from-literal=username=<citrix-adc-user> --from-literal=password=<citrix-adc-password> -n citrix-system

        helm repo add citrix https://citrix.github.io/citrix-helm-charts/

        helm install citrix-adc-istio-ingress-gateway citrix/citrix-cloud-native --namespace citrix-system --set iaIngress.enabled=true,iaIngress.ingressGateway.EULA=YES,iaIngress.secretName=nslogin,iaIngress.ingressGateway.netscalerUrl=https://<nsip>[:port],iaIngress.ingressGateway.vserverIP=<IPv4 Address>,iaIngress.ingressGateway.adcServerName=<ADC Cert Server Name>

## <a name="configuration-parameters">Configuration parameters</a>

The following table lists the configurable parameters in the Helm chart and their default values.



| Parameter                      | Description                   | Default                   | Optional/Mandatory                  |
|--------------------------------|-------------------------------|---------------------------|---------------------------|
| `iaIngress.enabled` | Mandatory | False | Set to "True" for deploying Citrix ADC as an Ingress Gateway in Istio environment. |
| `iaIngress.citrixCPX`                    | Citrix ADC CPX                    | FALSE                  | Mandatory for Citrix ADC CPX |
| `iaIngress.xDSAdaptor.image`            | Image of the Citrix xDS-adaptor container |quay.io/citrix/citrix-xds-adaptor:0.9.8 | Mandatory|
| `iaIngress.xDSAdaptor.imagePullPolicy`   | Image pull policy for xDS-adaptor | IfNotPresent       | Optional|
| `iaIngress.xDSAdaptor.secureConnect`     | If this value is set to true, Istio-adaptor establishes secure gRPC channel with Istio Pilot   | TRUE                       | Optional|
| `iaIngress.coe.coeURL`          | Name of [Citrix Observability Exporter](https://github.com/citrix/citrix-observability-exporter) Service in the form of "<servicename>.<namespace>"  | null            | Optional|
| `iaIngress.ADMSettings.ADMIP `          | Citrix Application Delivery Management (ADM) IP address  | NIL            | Mandatory for Citrix ADC CPX |
| `iaIngress.ADMSettings.ADMFingerPrint `          | Citrix Application Delivery Management (ADM) Finger Print. For more information, see [this](https://docs.citrix.com/en-us/citrix-application-delivery-management-service/application-analytics-and-management/service-graph.html)  | NIL            | Optional|
| `iaIngress.ADMSettings.licenseServerIP `          | Citrix License Server IP address  | NIL            | Optional |
| `iaIngress.ADMSettings.licenseServerPort` | Citrix ADM port if a non-default port is used                                                                                        | 27000                                                                 | Optional|
| `iaIngress.ADMSettings.bandWidth`          | Desired bandwidth capacity to be set for Citrix ADC CPX in Mbps  | NIL            | Optional |
| `iaIngress.ADMSettings.bandWidthLicense`          | To specify bandwidth based licensing  | false            | Optional |
| `iaIngress.ingressGateway.netscalerUrl`       | URL or IP address of the Citrix ADC which Istio-adaptor configures (Mandatory if citrixCPX=false)| null   |Mandatory for Citrix ADC MPX or VPX|
| `iaIngress.ingressGateway.vserverIP`       | Virtual server IP address on Citrix ADC (Mandatory if citrixCPX=false) | null | Mandatory for Citrix ADC MPX or VPX|
| `iaIngress.ingressGateway.adcServerName `          | Citrix ADC ServerName used in the Citrix ADC certificate  | NIL            | Optional |
| `iaIngress.ingressGateway.image`             | Image of Citrix ADC CPX designated to run as Ingress Gateway                                                                       |quay.io/citrix/citrix-k8s-cpx-ingress:13.0-76.29 |   Mandatory for Citrix ADC CPX                                                              |
| `iaIngress.ingressGateway.imagePullPolicy`   | Image pull policy                                                                                                                  | IfNotPresent                                                          | Optional|
| `iaIngress.ingressGateway.EULA`             | End User License Agreement(EULA) terms and conditions. If yes, then user agrees to EULA terms and conditions.                                     | NO                                                                    | Mandatory for Citrix ADC CPX 
| `iaIngress.ingressGateway.mgmtHttpPort`      | Management port of the Citrix ADC CPX                                                                                              | 9080                                                                  | Optional|
| `iaIngress.ingressGateway.mgmtHttpsPort`    | Secure management port of Citrix ADC CPX                                                                                           | 9443                                                                  | Optional|
| `iaIngress.ingressGateway.httpNodePort`      | Port on host machine which is used to expose HTTP port (80) of Citrix ADC CPX                                                       | 30180                                                                 |Optional|
| `iaIngress.ingressGateway.httpsNodePort`     | Port on host machine which is used to expose HTTPS port (443) of Citrix ADC CPX                                                     | 31443                                                                 |Optional|
| `iaIngress.ingressGateway.secretVolume`      | A map of user defined volumes to be mounted using Kubernetes secrets                                                               | null                                                                  |Optional|
| `iaIngress.ingressGateway.label` | Custom label for the Ingress Gateway service                                                                                       | citrix-ingressgateway                                                                 |Optional|
| `iaIngress.ingressGateway.tcpPort` | For exposing multiple TCP ingress                                                                                      | NIL                                                                 |Optional|
| `iaIngress.ingressGateway.netProfile `          | Network profile name used by [CNC](https://github.com/citrix/citrix-k8s-node-controller) to configure Citrix ADC VPX or MPX which is deployed as Ingress Gateway  | null            | Optional|
| `iaIngress.ingressGateway.multiClusterIngress `          | Flag indicating if Citrix ADC is acting as Ingress gateway to multi cluster Istio mesh installation. Possible values: true/false | false            | Optional|
| `iaIngress.ingressGateway.multiClusterListenerPort `          | Port opened on Citrix ADC to enable inter-cluster service to service (E-W) communication | 15443            | Optional|
| `iaIngress.ingressGateway.multiClusterListenerNodePort `          | Nodeport for multiClusterListenerPort in case of Citrix ADC CPX acting as Ingress gateway  | 32443            | Optional|
| `iaIngress.ingressGateway.multiClusterSvcDomain `          | Domain suffix of remote service (deployed in other cluster) used in E-W communication | global            | Optional|
| `iaIngress.istioPilot.name`                 | Name of the Istio Pilot (Istiod) service                                                                                                        | istiod                                                           |Optional|
| `iaIngress.istioPilot.namespace`     | Namespace where Istio Pilot is running                                                                                        | istio-system                                                          |Optional|
| `iaIngress.istioPilot.secureGrpcPort`       | Secure GRPC port where Istiod (Istio Pilot) is listening (default setting)                                                                  | 15012                                                                 |Optional|
| `iaIngress.istioPilot.insecureGrpcPort`      | Insecure GRPC port where Istio Pilot is listening                                                                                  | 15010                                                                 |Optional|
| `iaIngress.istioPilot.SAN`                 | Subject alternative name for Istio Pilot which is the secure production identity framework for everyone (SPIFFE) ID of Istio Pilot                                                        | null |Optional|
| `iaIngress.metricExporter.required`          | Metrics exporter for Citrix ADC                                                                                                    | TRUE                                                                  |Optional|
| `iaIngress.metricExporter.image`             | Image of the Citrix ADC Metrics Exporter                                                                                   | quay.io/citrix/citrix-adc-metrics-exporter:1.4.6                             |Optional|
| `iaIngress.metricExporter.port`              | Port over which Citrix ADC Metrics Exporter collects metrics of Citrix ADC.                                                      | 8888                                                                  |Optional|
| `iaIngress.metricExporter.secure`            | Enables collecting metrics over TLS                                                                                                | YES                                                                    |Optional|
| `iaIngress.metricExporter.logLevel`          | Level of logging in Citrix ADC Metrics Exporter. Possible values are: DEBUG, INFO, WARNING, ERROR, CRITICAL                                       | ERROR                                                                 |Optional|
| `iaIngress.metricExporter.imagePullPolicy`   | Image pull policy for Citrix ADC Metrics Exporter                                                                                       | IfNotPresent                                                          |Optional|
| `iaIngress.certProvider.caAddr`   | Certificate Authority (CA) address issuing certificate to application                           | istiod.istio-system.svc                          | Optional |
| `iaIngress.certProvider.caPort`   | Certificate Authority (CA) port issuing certificate to application                              | 15012 | Optional |
| `iaIngress.certProvider.trustDomain`   | SPIFFE Trust Domain                         | cluster.local | Optional |
| `iaIngress.certProvider.certTTLinHours`   | Validity of certificate generated by xds-adaptor and signed by Istiod (Istio Citadel) in hours. Default is 30 days validity              | 720 | Optional |
| `iaIngress.certProvider.jwtPolicy`   | Service Account token type. Kubernetes platform supports First party tokens and Third party tokens.  | first-party-jwt | Optional |
| `iaIngress.secretName`   | Name of the Kubernetes secret holding Citrix ADC credentials | nslogin | Mandatory for Citrix ADC VPX/MPX |
