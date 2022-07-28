# Slurm memory\-based scheduling<a name="slurm-mem-based-scheduling-v3"></a>

Starting with version 3\.2\.0, AWS ParallelCluster supports Slurm memory\-based scheduling with the cluster configuration parameter [`SlurmSettings`](Scheduling-v3.md#Scheduling-v3-SlurmSettings) / [`EnableMemoryBasedScheduling`](Scheduling-v3.md#yaml-Scheduling-SlurmSettings-EnableMemoryBasedScheduling)\.

With `EnableMemoryBasedScheduling: true`, the Slurm scheduler tracks the amount of memory required by each job on each node and uses this information to schedule multiple jobs on the same compute node\. The sum of the amounts of memory required by jobs on a node can't be larger than the available node memory\. The scheduler prevents a job from using more memory than requested when the job was submitted\.

With `EnableMemoryBasedScheduling: false`, jobs might compete for memory on a shared node and cause job failures and `out-of-memory` events\.

**Warning**  
Slurm uses a power of 2 notation for its labels, such as MB, GB, and similar\. These labels should be interpreted as MiB and GiB, respectively\.

## Slurm configuration and memory\-based scheduling<a name="slurm-mem-based-scheduling-config-v3"></a>

With `EnableMemoryBasedScheduling: true`, Slurm sets the following Slurm configuration parameters:
+ [https://slurm.schedmd.com/slurm.conf.html#OPT_CR_CPU_Memory](https://slurm.schedmd.com/slurm.conf.html#OPT_CR_CPU_Memory) in the `slurm.conf`\. This option configures node memory to be a consumable resource in Slurm\.
+ [https://slurm.schedmd.com/cgroup.conf.html#OPT_ConstrainRAMSpace](https://slurm.schedmd.com/cgroup.conf.html#OPT_ConstrainRAMSpace) in the Slurm `cgroup.conf`\. With this option, a job's access to memory is limited to the amount of memory that the job requested when submitted\.

**Note**  
Several other Slurm configuration parameters can impact the behavior of the Slurm scheduler and resource manager when these two options are set\. For more information see the [Slurm Documentation](https://slurm.schedmd.com/documentation.html)\.

## Slurm scheduler and memory\-based scheduling<a name="slurm-mem-based-scheduling-scheduler-v3"></a>

**`EnableMemoryBasedScheduling: false` \(default\)**

When `EnableMemoryBasedScheduling` is set to false \(default configuration\), Slurm doesn't include memory as a resource in its scheduling algorithm and doesn't track the memory used by jobs\. Users can specify the `--mem MEM_PER_NODE` option to set the minimum amount of memory per node required by the job\. This forces the scheduler to choose nodes with a `RealMemory` value of at least `MEM_PER_NODE` when scheduling the job\.

For example, a user submits two jobs with `--mem=5GB`\. The jobs can run at the same time on a node with 8 GiB of memory, provided that their other requested resources are available, such as CPUs or GPUs\. The two jobs won't be scheduled on compute nodes with less than 5 GiB `RealMemory`\.

**Warning**  
When memory\-based scheduling is disabled, Slurm doesn't track the amount of memory used by jobs\. Jobs running on the same node might compete for memory resources and cause the other job to fail\.  
When memory\-based scheduling is disabled, we recommend that users don't specify the [https://slurm.schedmd.com/srun.html#OPT_mem-per-cpu](https://slurm.schedmd.com/srun.html#OPT_mem-per-cpu) or [https://slurm.schedmd.com/srun.html#OPT_mem-per-gpu](https://slurm.schedmd.com/srun.html#OPT_mem-per-gpu) options\. These options might cause behavior that differs from what's described in the [Slurm Documentation](https://slurm.schedmd.com/documentation.html)\.

**`EnableMemoryBasedScheduling: true`**

When `EnableMemoryBasedScheduling` is set to true, Slurm tracks the memory usage of each job and prevents jobs from using more memory than requested with the `--mem` submission options\.

Using the previous example, a user submits two jobs with `--mem=5GB`\. The jobs can't run at the same time on a node with 8 GiB of memory because the sum of the required amounts of memory is greater than the memory available on the node\.

With memory\-based scheduling enabled, `--mem-per-cpu` and `--mem-per-gpu` behave consistently with what's described in the Slurm documentation\. For example, a job is submitted with `--ntasks-per-node=2 -c 1 --mem-per-cpu=2GB`\. In this case, Slurm assigns the job a total of 4 GiB per node\.

**Warning**  
When memory\-based scheduling is enabled, we recommend that users include a `--mem` specification when submitting a job\. With the default Slurm configuration shipped with AWS ParallelCluster, if no memory option is included \(`--mem`, `--mem-per-cpu` or `--mem-per-gpu`\), Slurm assigns entire memory of the allocated nodes to the job, even if it requests only a portion of the other resources, such as CPUs or GPUs\. This effectively prevents node sharing until the job is finished because no memory is available to other jobs\. This happens because Slurm sets the memory per node for the job to [https://slurm.schedmd.com/slurm.conf.html#OPT_DefMemPerNode](https://slurm.schedmd.com/slurm.conf.html#OPT_DefMemPerNode) when no memory specifications are provided at job submission time\. The default value for this parameter is 0 and specifies unlimited access to a node’s memory\.  
Also, if multiple types of compute resources with different amounts of memory are available in the same queue, a job submitted without memory options might be assigned different amounts of memory on different nodes, depending on which nodes the scheduler makes available to the job\. Users can define a custom value for options, such as `DefMemPerNode` or [https://slurm.schedmd.com/slurm.conf.html#OPT_DefMemPerCPU](https://slurm.schedmd.com/slurm.conf.html#OPT_DefMemPerCPU), at the cluster or partition level in the Slurm configuration files to prevent this behavior\.

## Slurm `RealMemory` and AWS ParallelCluster `SchedulableMemory`<a name="slurm-mem-based-scheduling-realmemory-v3"></a>

With the Slurm configuration that's shipped with AWS ParallelCluster, Slurm interprets [RealMemory](https://slurm.schedmd.com/slurm.conf.html#OPT_RealMemory) to be the amount of memory per node available to jobs\. Starting with version 3\.2\.0, by default, AWS ParallelCluster sets `RealMemory` to 95 percent of the memory listed in [Amazon EC2 Instance Types](http://aws.amazon.com/ec2/instance-types) and returned by the Amazon EC2 API [DescribeInstanceTypes](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_DescribeInstanceTypes.html)\.

When memory\-based scheduling is disabled, the Slurm scheduler uses `RealMemory` to filter nodes when users submit a job with `--mem` specified\.

When memory\-based scheduling is enabled, the Slurm scheduler interprets `RealMemory` to be the maximum amount of memory available to jobs running on the compute node\.

The default setting might not be optimal for all instance types:
+ This setting might be higher than the amount of memory that nodes can actually access\. This can happen when compute nodes are small instance types\.
+ This setting might be lower than the amount of memory that nodes can actually access\. This can happen when compute nodes are large instance types and can lead to significant amounts of unused memory\.

You can use [`SlurmQueues`](Scheduling-v3.md#Scheduling-v3-SlurmQueues) / [`ComputeResources`](Scheduling-v3.md#Scheduling-v3-SlurmQueues-ComputeResources) / [`SchedulableMemory`](Scheduling-v3.md#yaml-Scheduling-SlurmQueues-ComputeResources-SchedulableMemory) to fine\-tune the value of `RealMemory` configured by AWS ParallelCluster for compute nodes\. To override the default, you can define a custom value for `SchedulableMemory` specifically for your cluster configuration\.

To check the amount of memory actually available on a compute node, run the `/opt/slurm/sbin/slurmd -C` command on the node\. This command returns the hardware configuration of the node, including the [https://slurm.schedmd.com/slurm.conf.html#OPT_RealMemory](https://slurm.schedmd.com/slurm.conf.html#OPT_RealMemory) value\. For more information, see [https://slurm.schedmd.com/slurmd.html#OPT_-C](https://slurm.schedmd.com/slurmd.html#OPT_-C)\.

To ensure that the compute node's operating system processes have sufficient memory, limit the memory available to jobs by setting the value for `SchedulableMemory` to lower than the `RealMemory` value returned by the `slurmd -C` command\.