# RHEL Container

Adapted from [official documentation](https://learn.microsoft.com/en-us/sql/linux/quickstart-sql-server-containers-azure?view=sql-server-ver16&tabs=oc)

## Prerequisites

1. OCP Cluster
2. Download [sqlcmd utility](https://learn.microsoft.com/en-us/sql/tools/sqlcmd/sqlcmd-download-install?view=sql-server-ver17&tabs=mac#download-and-install-sqlcmd-go)

## Steps

1. Create a project

```bash
oc new-project mssql
```

2. Set a SQL Server System Administrator password. You MUST follow this [policy](https://learn.microsoft.com/en-us/sql/relational-databases/security/password-policy?view=sql-server-ver16#password-complexity).

```bash
SQL_SA_PASSWORD=RedHat123OpenShift
```

3. Create a secret for the System Administrator password

```bash
oc create secret generic mssql --from-literal=MSSQL_SA_PASSWORD=$SQL_SA_PASSWORD
```

4. Create storage

```bash
oc create -f pvc.yaml
```

5. Create MS SQL Server container based on RHEL UBI

```bash
oc create -f mssql-rhel.yaml
```

6. Get URL to server

```bash
SQL_SERVER_URI=$(oc get svc mssql-deployment -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')
echo $SQL_SERVER_URI
```

7. Connect to server and test

```bash
sqlcmd -S $SQL_SERVER_URI -U sa -P $SQL_SA_PASSWORD

1> SELECT name, database_id, create_date
FROM sys.databases;
GO
```

