# APPENGINE

1. CONFIGURE AND WORK APPENGINE
 
      gcloud components install app-engine-python

2. DEPLOY APP
   
   gcloud app deploy app.yml

    --version to specify a custom version ID\n
    --project to specify the project ID to use for this app
    --no-promote TO DEPLOY THE APP WITHOUT ROUTING TRAFFIC TO IT

3. LOGS

  gcloud app logs read

  gcloud app logs tail

4. BROWSER

  gcloud app browser

5. STOP

  gcloud app versions stop v1 v2

7. AUTO SCALLING

 7.1 DYNAMIC

     7.1.1 AUTO_SCALLING

     automatic_scaling
   	  target_cpu_utilization: 0.65
   	  min_instances: 5
   	  max_instances: 100
   	  min_pending_latency: 30ms #default value
   	  max_pending_latency:  automatic
   	  max_concurrent_requests: 50

      7.1.2 BASIC_SCALLING

      basic_scaling
    	  max_instances: 10
    	  idle_timeout: 20m

 7.2 RESIDENT

      manual_scaling:
        instances: 7

8. SPLIT TRAFFIC (IP_ADDRESS, COOKIE AND RANDOM) - GOOGAPPUID cookie

  8.1 SHOW VERSIONS

    gcloud app versions list

  8.2 SPLIT

    gcloud app services set-traffic serv --splits version=.4,version=.6

    ex.: gcloud app services set-traffic default --splits 20210208t202955=.4,20210208t210644=.6 --split-by=cookie

# NETWORKS

1. CREATE NETWORK

    gcloud compute networks create ace-exam-vpc1 --subnet-mode=auto

2. CREATE A CUSTOM VPC

    gcloud compute networks create vpc-excluir --subnet-mode=custom

    gcloud compute networks subnets create subnet-excluir \
        --network=vpc-excluir \
        --region=us-west2 \
        --range=10.10.0.0/16 \
        --enable-private-ip-google-access \ (dynamic route - regional route)
        --enable-flow-logs

2. SHARED VPC1

 2.1 before assign the Shared VPC Admin role, which uses the descriptor roles/compute.xpnAdmin

  gcloud ORGANIZATIONS add-iam-policy-binding [ORG_ID] --member='user:[EMAIL]' --role="roles/compute.xpnAdmin"

  OR

  gcloud RESOURCE-MANAGER folders add-iam-policy-binding [FOLDER_ID] --member='user:[EMAIL_ADDRESS]' --role='roles/compute.xpnAdmin'

 2.2 Once YOU HAVE SET THE SHARED VPC ADMIN ROLE AT THE ORGANIZATION LEVEL, you can issue the shared-vpc command

   gcloud compute shared-vpc enable [HOST_PROJECT_ID]

 2.3 Now that the shared VPC is created, YOU CAN ASSOCIATE PROJECTS using the gcloud compute shared-vpc associate-projects command.

  gcloud compute shared-vpc associated-projects add [service_project_id] --host-project [host_project_id]

3. PEERING

[REDE A PARA REDE B]
gcloud compute networks peerings create peer-project-excluir \
--network networkA \
--peer-project ace-projectB \
--peer-network networkB \
--auto-create-routes

[E O INVERSO AGORA. REDE B PARA REDE A]
gcloud compute networks peerings create peer-project-excluir \
--network networkB \
--peer-project ace-projectA \
--peer-network networkA \
--auto-create-routes

4. FIREWALL RULES

  gcloud compute firewall-rules create <NAME> --network=<NET-WORKNAME> --allow

  ex.: gcloud compute firewall-rules create ace-rule-excluir2 --network=vpc-excluir --allow tcp:20000-25000

# CLOUD DNS

1. CREATE DNS

  1.1 VISIBILITY - PRIVATE (not optional: networks, description)

    gcloud dns managed-zones create mydns --dns-name=myquadroszone.com --description=test --visibility=private --networks=default

  1.2 VISIBILITY PUBLIC (not optional: dnssec-state)

    gcloud dns managed-zones create mydns --dns-name=mydns.com --description=teste --visibility=public --dnssec-state=on

2. Add registries on DNS

    2.1 TYPE=A

    gcloud dns record-sets transaction start --zone=[dns-zone-name]
    gcloud dns record-sets transaction add 192.168.0.1 --name=[meuendereco].zone.com --type=A --ttl=300 --zone=[dns-zone-name]
    gcloud dns record-sets transaction execute --zone=[dns-zone-name]

    2.2 TYPE=CNAME
    gcloud dns record-sets transaction start --zone=[dns-zone-name]
    gcloud dns record-sets transaction add "server.example.com" "teteu.myquadroszone.com" --name=myquadroszone.com. --ttl=14400 --type=CNAME --zone=MANAGED_ZONE
    gcloud dns record-sets transaction execute --zone=[dns-zone-name]

# MANING IP (CLASSLESS)

1. expanding CIDR blocks

    gcloud compute networks subnets expand-ip-range sub-excluir --region=us-west2 --prefix-length=12

    Obs.: You cannot decrease them, though. You would have to re-create the subnet with a smaller number of addresses.

    ex.: if you have a /12 you cannot resize to /20

    gcloud compute networks subnets expand-ip-range subnet-excluir --prefix-lenght 20

# CONFIGURING ACCESS AND SECURITY (IAM, SERVICE ACCOUNT)

1. List IAM ROLES

    gcloud iam roles list --filter="name ~ roles/run"

2. Show projetc assign roles associate to users and sa

    gcloud project get-iam-policies <project-name> list

3. Associate a member to role in a project (policies)
                                                           (can be serviceaccount:)
    gcloud project add-iam-policy-binding <project-name> --member=user:user@gmail.com --role=roles/ ...

    ex.:

    gcloud projects add-iam-policy-binding cgp-enginer-certified --member=user:user@gmail.com --role=roles/compute.networkUser

4. show permissions associate a role

    gcloud iam roles describe <role-name>

**important**

Roles is a collections of permissions. To see the permissions associate to a determined roles, use describe commands

    ex: gcloud iam roles describe rolesappengine.appCreator

includedPermissions:
- appengine.applications.create
- resourcemanager.projects.get
- resourcemanager.projects.list

5. To create custom roles, you need know what Permissions you wanna associate with you custom role.

    gcloud iam roles create MY_ROLE_EXCLUIR --project=cgp-enginer-certified --permissions=appengine.applications.create,resourcemanager.projects.get

# MANAGING SERVICE ACCOUNTS

1. To configure *ACCESS CONTROLS FOR A VM*, you will need to configure *BOTH IAM ROLES AND SCOPES*. Scopes *ARE PERMISSIONS GRANTED TO A VM* to perform some operation.
   A scope is specified using a URL that starts with https://www.googleapis.com/auth/

   ex.: https://www.googleapis.com/auth/bigquery.insertdata

   An instance can only perform operations *ALLOWED* by both *IAM ROLES ASSIGNED* to the *SERVICE ACCOUNT* and *SCOPES* defined on the instance.
   if a role grants *ONLY READ-ONLY ACCESS* to cloud storage but a *scope ALLOWS WRITE ACCESS*, then *THE INSTANCE WILL NOT BE ABLE TO WRITE TO CLOUD STORAGE*

2. Scopes options:

    2.1 *ALLOW DEFAULT ACCESS*: Default access is usually sufficient
    2.2 *ALLOW FULL ACCESS TO ALL CLOUD APIS*: If you are not sure what to set, you can choose Allow Full Access, but be sure to assign IAM roles
        to limit what the instance can do.
    2.3 *SET ACCESS FOR EACH API*: If you want to choose scopes individually, choose Set Access For Each *API*

3. gcloud compute instances set-service-account - set service account and scopes for a Compute Engine instance

   gcloud compute instances set-service-account example-instance \
           --scopes=pubsub,trace --zone=us-central1-b \
           --service-account=example-account

# BUCKET

1. enable version on bucket

    gsutil versioning set on gs://<bucket_path>

    Obs.: bucket_path not bucket_path with file

2. change permissions on bucket

    gsutil acl ch -u john.doe@example.com:WRITE gs://example-bucket

    gsutil acl ch -u AllUsers:W gs://example-bucket

***OBS***.:
  CH ROLES
  You may specify the following roles with either their shorthand or
  their full name:

    R: READ
    W: WRITE
    O: OWNER


3. There may be times, however, when you would like to manually change a bucket’s storage class. In those cases,
  you can use the GSUTIL REWRITE command and SPECIFY THE -S FLAG

  gsutil rewrite -s [STORAGE_CLASS] gs://[PATH_TO_OBJECT]

# METADATA  

1. Crate a instance with metadata

    gcloud compute instaces create my_instance --metadata key=value

2. set metadata withim instance

    gcloud compute instances add-metadata <instance_name> --metadata key1=value1,key2=value2

3. get metadata for virtual machine (withim)

    http://metadata.google.internal/computeMetadata/v1/

***OBS**.: You cannot query an instance's default metadata from another instance or directly from your local computer.

***IMPORTANT***
to querying metadata,  you must provide the following HEADER in all request

Metadata-Flavor: Google

Ex.:
curl "http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/123456789-compute%40developer.gserviceaccount.com/?query_path=https%3A%2F%2Flocalhost%3A8200%2Fexample%2Fquery&another_param=true" -H "Metadata-Flavor: Google"

# CLOUDSQL

1. Backup onDemand

    gcloud cloud sql backups create <instance_name>

2. auto Backup

    gcloud sql instances patch <instance_name> --backup-start-time 01:00

3 remove auto backups

    gcloud sql instances patch <instance_name> --no-backup

# DATASTORE

1. Entity it's like Schemas on sqldatabase

2. create backups

  2.1 create bucket

      gsutil mb -p <project_id> -l <location/zone> gs://<bucket_name>

  2.2 export backup

      gcloud datastore export --namespace=<namespace> gs://<bucket_name>

  2.3 recover backup

      gcloud datastore import --namespace=<namespace> gs://<bucket_name>/<datastore_name>/<datastore_name>.overall_export_metadata

# BIGQUERY

1. estimate of HOW MUCH DATA WILL BE SCANNED ( --dry-run)

    bq query --use_legacy_sql=false --dry_run "select name, gender,sum(number) AS total from bigquery-public-data.usa_names.usa_1910_2013 group by name, gender order by total desc limit 10"

2. Jobs in BigQuery are processes used to LOAD, EXPORT, COPY, and QUERY data. Jobs are automatically created when you start any of these operations.

3. Show all jobs in project

    bq ls -j -a

4. describe job

    bq show --format=prettyjson --location=<location> -j <job_id>

# SPANNER

1. List Instances

   gcloud spanner instances List

2. List databases in instance

   gcloud spanner databases list --instance=desenv-excluir

3. Backup by instance/databases

   gcloud spanner backups create backup-excluir --instance=desenv-excluir --database=example-db --expiration-date=2021-02-17T10:49:41Z --async

4. Show backups

   gcloud spanner backups list  --instance=desenv-excluir

5. delete backups

   gcloud spanner backups delete backup-excluir --instance=desenv-excluir

6. delete database

   gcloud spanner databases delete example-db --instance=desenv-excluir

7. delete instance

    gcloud spanner instances delete desenv-excluir

# PUBSUB

**Obs***.: Subscriptions *CAN BE PULLED*, in which the *APPLICATION READS FROM A TOPIC*, or *PUSHED*, in which the *SUBSCRIPTION WRITES MESSAGES TO AN ENDPOINT*.
         If you want to use a push subscription, you will need to specify the URL of an endpoint to receive the message.

1. create topic :

    gcloud pubsub topics create mytopic

2. create subscriptions with filter

    gcloud pubsub subscriptions create sub-x --topic=mytopic --message-filter='attributes.domain="x"'

3. publish message

    gcloud pubsub topics publish mytopic --attribute=domain="y" --message="message7"

4. pull message on subscrition

    gcloud pubsub subscriptions pull sub-x --auto-ack

# BIGTABLE

1. create instance / clusters

    gcloud bigtable instances create instance-big-excluir --display-name=instance-big-excluir  --cluster-config=id=my-cluster-excluir,zone=us-east1-c

2. configure ENVIRONMENT

    echo instance=instance-big-excluir >> ~/.cbtrc

3. create table

    cbt createtable excluir

4. Tables contain columns, BUT BIGTABLE also has a concept of COLUMN FAMILIES. To create a column family called COLFAM1, use the following command

    cbt createfamily <table> <columns_family_name>

5. TO SET a value in family name

    cbt set <table> <row> family:[column]=val[@ts]
    ex.: cbt set excluir row1 columnf:col1=vamos_excluir

6. display content of the table

    cbt read <table>

7. remove instance

    gcloud bigtable instances delete <instance>

# SHOW QUOTAS (STATIC IP ETC)

1. gcloud compute project-info describe --project <name_of_project>

# CLOUD COMPUTE

***SNAPSHOT (to work with snapshots, a user must be assigned the COMPUTE STORAGE ADMIN ROLE)***

***If you are making a SNAPSHOT OF A DISK ON A WINDOWS SERVER, check the ENABLE VSS BOX to create an application-consistent snapshot WITHOUT HAVING TO SHUT DOWN THE INSTANCE. ***

1. Creating a snapshot schedule

    gcloud compute resource-policies create snapshot-schedule my-schedule \   <--- *IMPORTANT. ACTION before THE TYPE OF ACTION:*
    --description "my schedule" \
    --max-retention-days 1 \
    --start-time 22:00 \
    --hourly-schedule 12 \
    --region southamerica-east1    <--- *IMPORTANT*

2. Binding resource-policies snapshot with vm

    gcloud compute instances create <instance_name> --resource-policies= <resource-policies-snapshot>

    ex.:
    gcloud compute instances create my_instance --zone=zona --preemptible --machine-type=<machine_type> --resource-policies=my-schedule

3. binding with disk

    gcloud compute disks create <DISK_NAME> --resource-policies=policy

4. create a snapshot

    gcloud compute DISKS snapshot DISK-NAME --snapshot-names=SNAPSHOT-NAME --zone ZONE

# IMAGES

***The difference is that SNAPSHOTS are used to make data available on a disk, while IMAGES are used to create VM***

***you can create images from snapshots, disks, images and cloudstorage***

***Family allows you to group images. When a family is specified, the latest, nondeprecated image in the family is used.***

***Google’s deprecated images are available for use BUT MAY NOT BE PATCHED FOR SECURITY FLAWS OR OTHER UPDATES***

1. create image by disks

    gcloud compute images create <image_name> --source-image=<image_name>

2. by disk

    gcloud compute images create <image_name> --source-disk=<disk_name>

3. by bucket

    gcloud compute images create <image_name> --source-uri=<uri>

4. export by uri

    gcloud compute images export --destination-uri=<uri> --image=<image_name>

# DISKS

1. keep the disk after delete instance

    gcloud compute instances delete <instance_name> --zone <zones> --keep-disks=all

    Options:
        all  - All disk types.
        boot - The first partition is reserved for the root filesystem.
        data - A non-boot disk.

2. delete disk after delete instance

    gcloud compute instances delete ch06-instance-1 --zone us-central2-b --delete-disks=data

3. list machine types

    gcloud compute machine-types list --zones=<zone>

4. create instance

    gcloud compute instances create instance-excluir --zone=southamerica-east1-a --preemptible --machine-type=n1-standard-1 --labels="excluir=true"

5. filter by LABELS

    gcloud projects list --format="json"  --filter="labels.env=test AND labels.version=alpha"

6. set region

    gcloud config set compute/region REGION

7. create disks from snapshots

    gcloud compute disks create <DISK_NAME> --source-snapshot=<snap_shoot_name>

# INSTANCE GROUP

1. create instance template

    gcloud compute instance-templates create instance-template-excluir-v1 --source-instance=vm-template-instece-excluir --source-instance-zone=southamerica-east1-a
 
2. Instance groups can contain instances in a *SINGLE ZONE OR ACROSS A REGION*. The first *IS CALLED A ZONAL MANAGED INSTANCE GROUP*, and the second is called a regional *MANAGED INSTANCE GROUP. REGIONAL MANAGED INSTANCE GROUPS ARE RECOMMENDED BECAUSE THAT CONFIGURATION SPREADS THE WORKLOAD ACROSS ZONES, INCREASING RESILIENCY*.

3. create instance group by instance templates

    gcloud compute instance-groups managed create instance-groups-excluir --zone=southamerica-east1-a --template=instance-template-excluir-v1 --size=2

