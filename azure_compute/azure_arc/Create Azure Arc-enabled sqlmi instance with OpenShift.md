#### Create Azure Arc-enabled sqlmi instance with OpenShift

##### Microsoft official doc reference

- []: https://docs.microsoft.com/en-us/azure/azure-arc/data/create-data-controller-using-kubernetes-native-tools

- []: https://docs.microsoft.com/en-us/azure/azure-arc/data/create-data-controller-using-kubernetes-native-tools#overview

##### Prerequisites

- `kubectl` or `oc` command installed on your local machine

  - **Install kubectl**: 

    []: https://kubernetes.io/docs/tasks/tools/

  - **Install oc**: 

    []: https://mirror.openshift.com/pub/openshift-v4/clients/ocp/stable/

##### Create an Azure Red Hat OpenShift cluster with `azure-cli`

- Setup guide: 

  []: https://docs.microsoft.com/en-us/azure/openshift/tutorial-create-cluster

- Create `rbac`authentication with `azure-cli`:

  ```bash
  # Create rbac authentication for oc
  > az ad sp create-for-rbac --name jason-azarc  --role Contributor --scopes /subscriptions/<subscription ID>/resourceGroups/<group Name>
  {
    "appId": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
    "displayName": "jason-azarc",
    "password": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
    "tenant": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
  }
  # write down the appID and password
  ```

##### Connect to an Azure Red Hat OpenShift cluster

Official tutorial: 

[]: https://docs.microsoft.com/en-us/azure/openshift/tutorial-connect-cluster

```bash
# list console login info
az aro list-credentials --name ocluster --resource-group az-arc
# list azure openshift cluster info
PS C:\Users\hubo> az aro list -o table
Name      ResourceGroup    Location    ProvisioningState    WorkerCount    URL
--------  ---------------  ----------  -------------------  -------------  ----------------------------------
ocluster  az-arc           eastus      Succeeded            4              https://console-openshift-console.xxxxxxx.io/
# login in openshift
[root@azk8s-oc ~]# oc login <API URL> -u kubeadmin -p <password>
Login successful.
You have access to 68 projects, the list has been suppressed. You can list all projects with 'oc projects'

Using project "default".
Welcome! See 'oc help' to get started
```

##### Create a namespace for data controller

```bash
[root@azk8s-oc ~]# oc create namespace arc
namespace/arc created
# if use openshift cluster, must edit namespace 
[root@azk8s-oc ~]# oc edit namespace arc
namespace/arc edited
...
openshift.io/sa.scc.supplemental-groups: 1000700001/10000
openshift.io/sa.scc.uid-range: 1000700001/10000
...
```

##### Create the custom resource definitions

[]: https://docs.microsoft.com/en-us/azure/azure-arc/data/create-data-controller-using-kubernetes-native-tools#create-the-custom-resource-definitions

```bash
[root@azk8s-oc arc]# oc create -f https://raw.githubusercontent.com/microsoft/azure_arc/main/arc_data_services/deploy/yaml/custom-resource-definitions.yaml
[root@azk8s-oc arc]# oc project arc
Now using project "arc" on server "https://xxxxxxxxxxxxxxxxx".
[root@azk8s-oc arc]# oc apply -f arcdata-deployer.yaml
```

##### Create the bootstrapper service

[]: https://docs.microsoft.com/en-us/azure/azure-arc/data/create-data-controller-using-kubernetes-native-tools#create-the-bootstrapper-service

> Make sure image version is `v1.8.0_2022-06-14`, latest version `v1.9.0_2022-07-12` have pull issues

```bash
[root@azk8s-oc arc]# oc create -f https://raw.githubusercontent.com/microsoft/azure_arc/main/arc_data_services/deploy/yaml/bootstrapper.yaml
[root@azk8s-oc arc]# oc get pod
```

##### Create secrets for the metrics and logs dashboards

[]: https://docs.microsoft.com/en-us/azure/azure-arc/data/create-data-controller-using-kubernetes-native-tools#create-secrets-for-the-metrics-and-logs-dashboards

```bash
[root@azk8s-oc arc]# wget https://raw.githubusercontent.com/microsoft/azure_arc/main/arc_data_services/deploy/yaml/controller-login-secret.yaml
[root@azk8s-oc arc]# echo sql | base64 && echo Passw0rd |base64
c3FsCg==
UGFzc3cwcmQK
[root@azk8s-oc arc]# vim controller-login-secret.yaml
apiVersion: v1
data:
  password: UGFzc3cwcmQK
  username: c3FsCg==
kind: Secret
metadata:
  name: metricsui-admin-secret
type: Opaque
---
apiVersion: v1
data:
  password: UGFzc3cwcmQK
  username: c3FsCg==
kind: Secret
metadata:
  name: logsui-admin-secret
type: Opaque
```

##### Create the webhook deployment job, cluster role and cluster role binding

[]: https://docs.microsoft.com/en-us/azure/azure-arc/data/create-data-controller-using-kubernetes-native-tools#create-the-webhook-deployment-job-cluster-role-and-cl

```bash
[root@azk8s-oc arc]# wget https://raw.githubusercontent.com/microsoft/azure_arc/main/arc_data_services/deploy/yaml/web-hook.yaml
# Edit the file and replace {{namespace}} in all places with the name of the namespace you created in the previous step
[root@azk8s-oc arc]# vim web-hook.yaml
[root@azk8s-oc arc]# oc create -f web-hook.yaml
```

##### Create the data controller

[]: https://docs.microsoft.com/en-us/azure/azure-arc/data/create-data-controller-using-kubernetes-native-tools#create-the-data-controller

```bash
[root@azk8s-oc arc]#  wget https://raw.githubusercontent.com/microsoft/azure_arc/release-arc-data/arc_data_services/deploy/yaml/data-controller.yaml
# replace some value base on your own env 
[root@azk8s-oc arc]# vim data-controller.yaml
[root@azk8s-oc arc]# oc create -f data-controller.yaml
[root@azk8s-oc mnt]# oc get pod
NAME                 READY   STATUS    RESTARTS        AGE
bootstrapper-pf2kn   1/1     Running   0               3h32m
control-swcxt        2/2     Running   1 (3h19m ago)   3h21m
controldb-0          2/2     Running   0               3h21m
logsdb-0             3/3     Running   0               3h20m
logsui-pq4ps         3/3     Running   0               3h19m
metricsdb-0          2/2     Running   0               3h20m
metricsui-c4bxg      2/2     Running   0               3h20m
```

##### Create sqlmi instance with Microsoft office template

[]: https://github.com/microsoft/azure_arc/blob/main/arc_data_services/deploy/yaml/sqlmi.yaml

```yaml
[root@azk8s-oc mnt]# wget https://raw.githubusercontent.com/microsoft/azure_arc/main/arc_data_services/deploy/yaml/sqlmi.yaml
[root@azk8s-oc mnt]# vim sqlmi.yaml
apiVersion: v1
data:
  password: <your base64 encoded password>
  username: <your base64 encoded username> 
	...
spec:
  dev: true #options: [true, false]
  licenseType: LicenseIncluded #options: [LicenseIncluded, BasePrice].  BasePrice is used for Azure Hybrid Benefits.
  tier: GeneralPurpose #options: [GeneralPurpose, BusinessCritical]
  ...
  services:
    primary:
      type: LoadBalancer # base on your env
  storage:
    data:
      volumes:
      - className: default #  use oc get storageclasses
        size: 5Gi
    datalogs:
      volumes:
      - className: default # oc get storageclasses
        size: 5Gi
    logs:
      volumes:
      - className: default # oc get storageclasses
        size: 5Gi
```

```bash
[root@azk8s-oc mnt]# oc get sqlmi
NAME   STATUS   REPLICAS   PRIMARY-ENDPOINT   AGE
sql1   Ready    2          10.0.1.5,31477     3h26m
[root@azk8s-oc mnt]# sqlcmd -S 10.0.1.5,31477 -U <username> -P <Password>
```







