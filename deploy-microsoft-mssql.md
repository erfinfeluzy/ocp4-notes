# Deploy Microsoft SQL Server 2017 (mssql) on Openshift 4

### Step 1: Create new project
```bash
$ oc new-project mssql
```

### Step 2: Create MSSQL Secret for MSSQL password
```bash
$ oc create secret generic mssql --from-literal=SA_PASSWORD="Sql2019isfast"
```

### Step 3: Create PVC for persistent volume
```bash
$ oc apply -f https://raw.githubusercontent.com/erfinfeluzy/ocp4-notes/master/assets/mssql-storage.yaml
```

### Step 4: Deploy MS SQL SErver 2019
```bash
$ oc apply -f https://raw.githubusercontent.com/erfinfeluzy/ocp4-notes/master/assets/mssql-deployment.yaml
```
> note: 
- deployment uses image from microsoft **mcr.microsoft.com/mssql/server:2017-CU8-ubuntu**
- expose service as loadbalancer

### Step 6: Query on Pods

Remote shell to mssql pods
```bash
$ oc get pods

$ oc rsh mssql-deployment-XXXXXXX
```
query inside pods
```bash
$ cd /opt/mssql-tools/bin

$ ./sqlcmd -S localhost -U sa -P Sql2019isfast

1> create database dbtest
2> select name from sys.databases
3> go
name                                                                                                                    
------------------------------
master                                                                                                                  
tempdb                                                                                                                  
model                                                                                                                   
msdb                                                                                                                    
dbtest
(5 rows affected)

1> use dbtest
2> create table tphone(id int, name nvarchar(50))
3> insert into tphone values(100,'Aina')
4> go
Changed database context to 'dbtest'.
(1 rows affected)

1> select * from tphone
2> go
id          name
----------- --------------------------------------------------
        100 Aina
(1 rows affected)
1> quit
```

### Additional: Deploy .NetCore Sample Application
```bash
$ oc apply -f https://raw.githubusercontent.com/erfinfeluzy/ocp4-notes/master/assets/dotnet-template-erfin.json
```
#### select template
![Select Template](https://github.com/erfinfeluzy/ocp4-notes/blob/master/screenshot/dotnet-select-template.png?raw=true)

#### instantiate template
![Instantiate](https://github.com/erfinfeluzy/ocp4-notes/blob/master/screenshot/dotnet-instantiate-template.png)

#### Check database

```bash
$ oc get pods

$ oc rsh mssql-deployment-XXXXXXX
```
Query inside mssql pods
```bash
$ cd /opt/mssql-tools/bin
$ ./sqlcmd -S localhost -U sa -P Sql2019isfast
1> use myContacts
2> go
Changed database context to 'myContacts'.
1> select * from Customers
2> go
Id          Name
----------- ----------------------------------------------------------------------------------------------------
          1 erfin2
          2 aryo

(2 rows affected)
```
#### Additional: Change icon dotnet and mssql
Add label to mssql deployment
```bash
app.kubernetes.io/name=mssql
```
Add label to dotnet deployment
```bash
app.kubernetes.io/name=dotnet
```
