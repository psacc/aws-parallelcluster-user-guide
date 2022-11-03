# Slurm accounting with AWS ParallelCluster<a name="slurm-accounting-v3"></a>

Starting with version 3\.3\.0, AWS ParallelCluster supports Slurm accounting with the cluster configuration parameter [SlurmSettings](Scheduling-v3.md#Scheduling-v3-SlurmSettings) / [Database](Scheduling-v3.md#Scheduling-v3-SlurmSettings-Database)\.

With Slurm accounting, you can integrate an external accounting database to:
+ Manage cluster users or groups of users, along with other entities\. This enables the use of Slurm's more advanced features, such as resource limit enforcement, fairshare, QOSs, and others\.
+ Collect and save job data, such as which user ran the job, the job's duration, the resources it uses, and other details\. You can view the saved data with the `sacct` utility\.

**Note**  
AWS ParallelCluster supports Slurm accounting for [Slurm supported MySQL database servers](https://slurm.schedmd.com/accounting.html#mysql-configuration)\.

## Working with Slurm accounting in AWS ParallelCluster<a name="slurm-accounting-works-v3"></a>

Before you configure Slurm accounting, you must have an existing external database server and database that uses `mysql` protocol\.

To configure Slurm accounting with AWS ParallelCluster, you must define:
+ The URI for the external database server in [Database](Scheduling-v3.md#Scheduling-v3-SlurmSettings-Database) / [Uri](Scheduling-v3.md#yaml-Scheduling-SlurmSettings-Database-Uri)\. The server must exist and be reachable from the head node\.
+ Credentials to access the external database defined in [Database](Scheduling-v3.md#Scheduling-v3-SlurmSettings-Database) / [PasswordSecretArn](Scheduling-v3.md#yaml-Scheduling-SlurmSettings-Database-PasswordSecretArn) and [Database](Scheduling-v3.md#Scheduling-v3-SlurmSettings-Database) / [UserName](Scheduling-v3.md#yaml-Scheduling-SlurmSettings-Database-UserName)\. AWS ParallelCluster uses this information to configure accounting at the Slurm level and the `slurmdbd` service on the head node\. `slurmdbd` is the daemon that manages communication between the cluster and the database server\.

To step through a tutorial, see [Creating a cluster with Slurm accounting](tutorials_07_slurm-accounting-v3.md)\.

**Note**  
AWS ParallelCluster performs a basic bootstrap of the Slurm accounting database by setting the default cluster user as database admin in the Slurm database\. AWS ParallelCluster doesn't add any other user to the accounting database\. The customer is responsible for managing the accounting entities in the Slurm database\.

AWS ParallelCluster configures [https://slurm.schedmd.com/slurmdbd.html](https://slurm.schedmd.com/slurmdbd.html) to ensure that a cluster has its own Slurm database on the database server\. The same database server can be used across multiple clusters, but each cluster has its own separate database\. AWS ParallelCluster uses the cluster name to define the name for the database in the `slurmdbd` configuration file [https://slurm.schedmd.com/slurmdbd.conf.html#OPT_StorageLoc](https://slurm.schedmd.com/slurmdbd.conf.html#OPT_StorageLoc) parameter\. If a database that's present on the database server, includes a cluster name that doesn't map to an active cluster name, you can create a new cluster with that cluster name to map to that database\. Slurm reuses it for the new cluster\.

**Warning**  
We recommend that you don't set up more than one cluster to use the same database at the same time\. This configuration can lead to potential performance issues or even deadlock situations on the database\.
Slurm accounting can add strain on the head node of the cluster\. We recommend that you choose an instance type with a more powerful CPU, more memory and higher network bandwidth for the head node if Slurm accounting is enabled\.

In the current architecture of the AWS ParallelCluster Slurm accounting feature, each cluster has its own instance of the `slurmdbd` daemon as shown in the following diagram example configurations\.

 ![\[\]](http://docs.aws.amazon.com/parallelcluster/latest/ug/images/slurm-acct-arch.png) 

If you're adding custom Slurm multi\-cluster or federation functionalities to your cluster environment, all clusters must reference the same `slurmdbd` instance\. For this alternative, we recommend that you enable AWS ParallelCluster Slurm accounting on one cluster and manually configure the other clusters to connect to the `slurmdbd` hosted on the first cluster\.

If you're using AWS ParallelCluster versions prior to version 3\.3\.0, learn about an alternate method to implement Slurm accounting in this [HPC Blog Post](https://aws.amazon.com/blogs/compute/enabling-job-accounting-for-hpc-with-aws-parallelcluster-and-amazon-rds/)\.

## Slurm accounting considerations<a name="slurm-accounting-considerations-v3"></a>

### Database and cluster on different VPCs<a name="slurm-accounting-considerations-different-vpcs-v3"></a>

To enable Slurm accounting, a database server is needed to serve as a backend for the read and write operations performed by the `slurmdbd` daemon\. The head node must be able to reach the database server before the Slurm accounting feature is enabled when the cluster created or updated\.

If you need to deploy the database server on a VPC, other than the one used by the cluster, consider the following:
+ To enable communication between the `slurmdbd` on the cluster side and the database server, you must set up connectivity between the two VPCs\. For more information, see [VPC Peering](https://docs.aws.amazon.com/vpc/latest/peering/what-is-vpc-peering.html) in the *Amazon Virtual Private Cloud User Guide*\.
+ The security group to be attached to the head node must be created on the VPC of the cluster\. The cross\-linking between the database side and the cluster side security groups is available after the two VPCs have been peered\. For more information, see [Security Group Rules](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_SecurityGroups.html#SecurityGroupRules) in the *Amazon Virtual Private Cloud User Guide*\.

### Configuring TLS encryption between `slurmdbd` and the database server<a name="slurm-accounting-considerations-tls-config-v3"></a>

With the default Slurm accounting configuration provided by AWS ParallelCluster, `slurmdbd` establishes a TLS encrypted connection to the database server, provided the server supports TLS encryption\. AWS database services such as Amazon RDS and Amazon Aurora support TLS encryption by default\.

You can require secure connections on the server side by setting the `require_secure_transport` parameter on the database server\. This is automatically configured in the provided CloudFormation template\.

To further enhance security, we recommend that you also enable server identity verification on the `slurmdbd` client\. To do this, configure the [StorageParameters](https://slurm.schedmd.com/slurmdbd.conf.html#OPT_StorageParameters) in the `slurmdbd.conf`\. Upload the server CA certificate to the head node of the cluster and set the [SSL\_CA](https://slurm.schedmd.com/slurmdbd.conf.html#OPT_SSL_CA) option of `StorageParameters` in `slurmdbd.conf` to the path of the server CA certificate on the head node\. This automatically enables server identity verification on the `slurmdbd` side\. After doing this, restart the `slurmdbd` service to re\-establish connectivity to the database server with identity verification enabled\.

### Updating the database credentials<a name="slurm-accounting-considerations-updates-v3"></a>

To update the values for [Database](Scheduling-v3.md#Scheduling-v3-SlurmSettings-Database) / [UserName](Scheduling-v3.md#yaml-Scheduling-SlurmSettings-Database-UserName) or [PasswordSecretArn](Scheduling-v3.md#yaml-Scheduling-SlurmSettings-Database-PasswordSecretArn) you must first stop the compute fleet\. When the secret value stored in the AWS Secrets Manager secret is changed, with no change to its ARN, the cluster doesn't automatically update the database password to the new value\. To update the cluster for the new secret value, you must run the following command from the head node\.

```
$ sudo /opt/parallelcluster/scripts/slurm/update_slurm_database_password.sh
```

**Warning**  
We recommend that you only change the database password when the compute fleet is stopped to avoid loss of accounting data\.

### Database Monitoring<a name="slurm-accounting-considerations-updates-v3"></a>

We recommend that you enable the monitoring functionalities made available by the AWS database services\. For more information, see [Amazon RDS monitoring](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_Monitoring.html) or [Amazon Aurora monitoring](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/MonitoringAurora.html) documentation\. 