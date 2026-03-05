## Prerequisites

1. OCP Cluster in AWS
2. Download [virtctl](https://docs.redhat.com/en/documentation/openshift_container_platform/4.21/html/virtualization/getting-started#installing-virtctl_virt-using-the-cli-tools)

## Steps

### Bare Metal Machine

1. Make a copy of existing MachineSet configuration:

```bash
MACHINESET=$(oc get machineset -n openshift-machine-api -o jsonpath='{.items[0].metadata.name}')
oc get machineset $MACHINESET -n openshift-machine-api -o yaml > scratch/baremetal-machineset.yaml
```

2. Edit `scratch/baremetal-machineset.yaml`

  - [ ] Delete `creationTimestamp`, `generation`, `resourceVersion`, `uid`
  - [ ] Set `.metadata.name` to `baremetal-machineset`
  - [ ] Set `.spec.replicas` to `1`
  - [ ] Set `.spec.selector.matchLabels["machine.openshift.io/cluster-api-machineset"]` to `baremetal-machineset`
  - [ ] Set `.spec.template.metadata.labels["machine.openshift.io/cluster-api-machineset"]` to `baremetal-machineset`
  - [ ] Set `.spec.template.spec.providerSpec.value.instanceType` to `m5zn.metal`

3. Create bare metal machine

```bash
oc create -f scratch/baremetal-machineset.yaml
```

### OpenShift Virtualization

1. Install [OpenShift Virtualization](https://docs.redhat.com/en/documentation/openshift_container_platform/4.21/html/virtualization/installing)

2. Create HyperConvered Resource (use default)

3. Create a project and ssh key

```bash
oc new-project mssql-virt
oc create secret generic authorized-keys --from-file=ssh-publickey=$HOME/.ssh/id_rsa.pub
```
4. Create a VM

```bash
oc create -f mssql-rhel-vm.yaml
```

5. Expose the VM

```bash
oc create -f mssql-virt-svc.yaml
```

### MS SQL Server

1. SSH into the VM

```bash
virtctl ssh cloud-user@vm/mssql-rhel-vm -i ~/.ssh/id_rsa -n mssql-virt
```

2. Register to subscription manager

```bash
sudo subscription-manager register
```

3. Install firewall

```bash
sudo yum install firewalld
sudo systemctl enable --now firewalld
```

4. Follow these steps to install SQL Server, [link](https://learn.microsoft.com/en-us/sql/linux/quickstart-install-connect-red-hat?view=sql-server-linux-ver17)

5. Get URL to server

```bash
SQL_SERVER_URI=$(oc get svc mssql-virt -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')
echo $SQL_SERVER_URI
```

6. Connect to server and test

```bash
sqlcmd -S $SQL_SERVER_URI -U sa -P $SQL_SA_PASSWORD

1> SELECT name, database_id, create_date
FROM sys.databases;
GO
```