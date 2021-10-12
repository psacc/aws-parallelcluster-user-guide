# `pcluster create-cluster`<a name="pcluster.create-cluster-v3"></a>

Creates an AWS ParallelCluster cluster\.

```
pcluster create-cluster [-h] [--region REGION]
                        [--suppress-validators SUPPRESS_VALIDATORS [SUPPRESS_VALIDATORS ...]]
                        [--validation-failure-level {INFO,WARNING,ERROR}]
                        [--dryrun DRYRUN]
                        [--rollback-on-failure ROLLBACK_ON_FAILURE]
                         --cluster-name CLUSTER_NAME
                         --cluster-configuration CLUSTER_CONFIGURATION
                        [--debug] [--query QUERY]
```

## Named arguments<a name="pcluster-v3.create-cluster.namedargs"></a>

`-h, --help`  
Shows the help text for `pcluster create-cluster`\.

`--region REGION`  
Specifies the AWS Region to use\. The Region must be specified, either using the [`Region`](cluster-configuration-file-v3.md#yaml-Region) setting in the cluster configuration file, the `AWS_DEFAULT_REGION` environment variable, or the `--region` parameter\.

`--suppress-validators SUPPRESS_VALIDATORS [SUPPRESS_VALIDATORS ...]`  
Identifies one or more config validators to suppress\.  
 Format: \(`ALL`\|type:`[A-Za-z0-9]+`\)

`--validation-failure-level {INFO,WARNING,ERROR}`  
Specifies the minimum validation level that will cause the creation to fail\. \(Defaults to `ERROR`\.\)

`--dryrun DRYRUN`  
Specifies that the command will only perform validation without creating any resources\. May be used to validate the cluster configuration\. \(Defaults to `false`\.\)

`--rollback-on-failure ROLLBACK_ON_FAILURE`  
When `true`, automatically initiates a cluster stack rollback on failures\. \(Defaults to `true`\.\)

`--cluster-name CLUSTER_NAME`  
Specifies the name of the cluster to be created\.

`--cluster-configuration CLUSTER_CONFIGURATION`  
Specifies the YAML cluster configuration file\.

`--debug`  
Enables debug logging\.

`--query QUERY`  
Specifies the JMESPath query to perform on the output\.