#### Azure-arc enabled SQLMI instance Password and Username Encode and Decode

##### Password and Username Encode and Decode on Linux

```bash
[root@azk8s-oc sqlmi]# head sql1-linux.yaml
apiVersion: v1
data:
  username: c3FsYWRtaW4K
  password: SHVhd2VpMTIjMjMK
kind: Secret
metadata:
  name: sql1-login-secret
type: Opaque
---
apiVersion: sql.arcdata.microsoft.com/v5
[root@azk8s-oc sqlmi]# echo sqladmin | base64
c3FsYWRtaW4K
[root@azk8s-oc sqlmi]# echo Huawei12#23 |base64
SHVhd2VpMTIjMjMK
[root@azk8s-oc sqlmi]# oc get sqlmi sql1
NAME   STATUS   REPLICAS   PRIMARY-ENDPOINT    AGE
sql1   Ready    1          20.232.50.69,1433   5m1s
[root@azk8s-oc sqlmi]# sqlcmd -S 20.232.50.69,1433 -Usqladmin -PHuawei12#23
Sqlcmd: Error: Microsoft ODBC Driver 17 for SQL Server : Login failed for user 'sqladmin'..
```

##### Password and Username Encode and Decode with Powershell

```powershell
[root@azk8s-oc sqlmi]# pwsh
PS /mnt/sqlmi> [Convert]::ToBase64String([System.Text.Encoding]::UTF8.GetBytes('sqladmin'))
c3FsYWRtaW4=
PS /mnt/sqlmi> [Convert]::ToBase64String([System.Text.Encoding]::UTF8.GetBytes('Huawei12#23'))
SHVhd2VpMTIjMjM=
```

```bash
[root@azk8s-oc sqlmi]# head sql2-powershell.yaml
apiVersion: v1
data:
  username: c3FsYWRtaW4=
  password: SHVhd2VpMTIjMjM=
kind: Secret
metadata:
  name: sql2-login-secret
type: Opaque
---
apiVersion: sql.arcdata.microsoft.com/v5
[root@azk8s-oc sqlmi]# oc get sqlmi sql2
NAME   STATUS   REPLICAS   PRIMARY-ENDPOINT      AGE
sql2   Ready    1          x.x.x.x,1433          5m10s
[root@azk8s-oc sqlmi]# sqlcmd -S <x.x.x.x>,1433 -Usqladmin -PHuawei12#23
1> select @@version;
2> go
Microsoft Azure SQL Managed Instance - Azure Arc - 16.0.41.7339 (X64)
        May 27 2022 11:38:57
        Copyright (C) 2021 Microsoft Corporation
        General Purpose (64-bit) on Linux (Ubuntu 20.04.4 LTS) <X64>
(1 rows affected)
1> exit
[root@azk8s-oc sqlmi]#

```

