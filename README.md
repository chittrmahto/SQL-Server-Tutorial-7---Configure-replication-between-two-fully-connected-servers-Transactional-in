# SQL-Server-Tutorial-7---Configure-replication-between-two-fully-connected-servers-Transactional-in
SQL Server Tutorial 7 - Configure replication between two fully connected servers (Transactional) in SQL Server


Steps: ---
I have two Servers in same zone and they are fully conneted.

**Prepare SQL Server for replication (publisher, distributor, subscriber)**

=Create windows accounts for replication =
list of the agents for replication in server 1 (publisher server)
<machine_name>\repl_snapshot
<machine_name>\repl_logreader
<machine_name>\repl_distribution
<machine_name>\repl_merge

list of the agents for replication in server 2 (distributor)
<machine_name>\repl_distribution
<machine_name>\repl_merge

^^create local Windows accounts for replication agents at the publisher in server 1(Publisher)^^
1. At the Publisher, Computer Management -> Administrative tools in control panel.
2. System Tools, expand Local Users and Groups.
3. Right click Users and then select New User.
<machine_name>\repl_snapshot
<machine_name>\repl_logreader
<machine_name>\repl_distribution
<machine_name>\repl_merge
all the user are created in publisher server 1.


^^ Create local Windows accounts for replication agents at the subscriber - Server 2 ^^
1. at the subscriber, Computer Management -> Administrative tools in control panel.
2. System Tools, expand Local Users and Groups.
3. Right click Users and then select New User.
These users are created in machine 2 (subsriber machine)
<machine_name>\repl_distribution
<machine_name>\repl_merge
users created successfully in both the servers.

Now move to Server 1/Machine 1/ Publisher Machine.

## Prepare the snapshot folder in publisher server ##
** create a share for the snapshot folder and assign permissions **
1. In File Explorer, browse to the SQL Server data folder. The default location is
C:\Program Files\Microsoft SQL Server\MSSQL.X\MSSQL\Data.

2. Crate foler named "repldata".
3. right-click this folder and select properties.
 a. Sharing->Advanced Sharing
 b. advanced Sharing -> share this folder->Permissions.
4. Add->Advanced->Find Now
select below list of the user.
<machine_name>\repl_snapshot
<machine_name>\repl_distribution
<machine_name>\repl_merge

5. After you add the three accounts, assign the following permissions:
- repl_distribution : Read
- repl_merge : Read
- repl_snapshot : Full Control

6. after your share permission.. close the permission for repldata dialog box.

7. In the repldata Properties->Security->Edit->Add->Advanced->Find Now.

8. add all three users
<machine_name>\repl_snapshot
<machine_name>\repl_distribution
<machine_name>\repl_merge

9. Verify that the following permissions are allowed :
- repl_distribution : Read
- repl_merge : Read
- repl_snapshot : Full Control

10. keep the sharing path for futher use.
\\SQLTEST01\repldata

"now repldata folder is empty."


### Configure Disribution in publisher ( server 1 ) ###
^^ configure distribution at the publisher ^^

1. connect SSMS (SQL Server Management Studio) with your credentials. (Login with Server name instant of server ip address-flow the video)
2. Right-click the Replication->Configure Distribution->Next->select radio (First Option) O-'SQLTEST01' will act as its own...... ->Next
3. put the shre path in snapshot folder box.
\\SQLTEST01\repldata
next->next->next->Finish->Close

Note: You might see the "SQL Server Agent" error. if SQL Server Agent is stoped the please start.


^^ Create a database or you should have the database which one you want to Replicate. ^^

Now i am creating database.
 CREATE DATABASE CHIRAG
 GO

 USE CHIRAG
 GO

 CREATE TABLE TEST
 (
  ID VARCHAR(10) PRIMARY KEY NOT NULL,
  NAME VARCHAR(10)
 )
 GO

Now insert some data into table.

 INSERT INTO TEST VALUES ('1','A')
 INSERT INTO TEST VALUES ('2','B')
 INSERT INTO TEST VALUES ('3','C')
 INSERT INTO TEST VALUES ('4','D')
 INSERT INTO TEST VALUES ('5','E')


^^ Set database permissions at the publisher ^^

1. SSMS -> Security ->(Right-click) Logins-> New Login->Search->Advanced->Find Now ->Follow Video.
add "repl_snapshot"

2. User Mapping->select both the databases (distribution and PUB) to db_owner.

3. Repeat Pervious steps for create a login for the other local accounts
 repl_distribution
 repl_logreader
 repl_merge

First part of work is DONE.


### Configure replication between two fully connected server (Transactional) ###

^^^ Configure the publisher for trnsactional replication ^^^

*Create a publication and define articles *
1. connect to the publisher SSMS. My publisher machine is SQLTEST01.
2. Start the SQL Server Agent(If not started)
3. Replication->Local Publications->New Publication
Next->(Select Database) Next->(Select Publicaation Type) Transactional publication->Next->select database tables->Next->Checkbox [] Create a snapshot immediately....(Select)->Next

Secutrity Settings->0 Run under the following Windows account:
SQLTEST01\repl_snapshot
Ok

Now uncheck the [] Use the security settings from the snapshot Agent

Log Reader Agent
Security Settings->Put "SQLTEST01\repl_logreader" then password then "Ok".

Next->(Publication Name in your choice) PUBTrans->Finish->Close.

Note: You might encounter the "SQL Server Agent" error. Start if "SQL Server Agent" is not Started.


^^ View the status of snapshot generation^^

1. Connect SSMS in Publisher
2. Local Publications -> (right-click) PUBTrans->View Snapshot Agent Status.

You ll get the last message
"[100%] A snapshot of 1 article(s) was generated."

^^^ Add the Distribution Agent login to the PAL (Publication Access List) ^^^
1. Connect SSMS in Publisher
2. Local Publications -> (right-click) PUBTrans->Properties->Publication Access List->Add


### Create a subscription to the transactional publication ###
^^ Create the subscription^^
1. Connect SSMS in Publisher
2. Local Publications -> (right-click) PUBTrans->New Subscriptions->Next->select Databases and publications->Next->Default Select->Next->Add Subscription->Add Sql Server Subscriber->Login with Subscriber Server credentials->Subscription Database-><New database> Ok->Subscription Database name "PUBSubs"->Next->Click on (...)->0 Run Under the following Windows Account:
SQLTEST01\repl_distribution
Ok->Next->.... Finish->Close.



### Set the permission at the subscriber Server 2###
1. Connect SSMS at Subscriber (Server 2).
Security->Logins->New Login->Search->Advanced->Find Now-> Select  repl_distribution.



### View the synchronization status of the subscription in Server 1 ###
1. Connect to the publisher SSMS.
2. Replication->Local Publications->PUBTrans (Expend)->(Right-click)SQLTEST02[PUBSubs]->View Synchronization Status.

No Error.

Now you can see the database is replicated into second server.


Now i ll check the transnational replication is working or not. Its Working :)

Just add or insert one more row into the Test Table.

Now i ll delete one row from server 1(Publisher)...its reflecting in server 2.

And the new row ll automatic reflect into another replicated database.


Video URL : https://chittranjanmahto.blogspot.com/2019/08/sql-server-tutorial-7-configure.html
