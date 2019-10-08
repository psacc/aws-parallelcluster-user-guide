# Elastic Fabric Adapter<a name="efa"></a>

Elastic Fabric Adapter \(EFA\) is a network device that has OS\-bypass capabilities for low\-latency network communications with other instances on the same subnet\. This is exposed using libfabric and can be used by applications using the Messaging Passing Interface \(MPI\)\. To use EFA with AWS ParallelCluster, add the line `enable_efa = compute` to the [[cluster] section](cluster-definition.md)\. EFA is supported by specific instance types \(`c5n.18xlarge`, `c5n.metal`, `i3en.24xlarge`, and `p3dn.24xlarge`\)\. For more information about the `enable_efa` setting, see [`enable_efa`](cluster-definition.md#enable-efa)\. A cluster placement group should be used to minimize latencies between instances\. See [`placement`](cluster-definition.md#placement) and [`placement_group`](cluster-definition.md#placement-group)\.

For more information, see [Elastic Fabric Adapter](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/efa.html) in the *Amazon EC2 User Guide for Linux Instances* and [Scale HPC Workloads with Elastic Fabric Adapter and AWS ParallelCluster](https://aws.amazon.com/blogs/opensource/scale-hpc-workloads-elastic-fabric-adapter-and-aws-parallelcluster/) in the *AWS Open Source Blog*\.