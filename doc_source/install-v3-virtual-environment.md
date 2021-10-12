# Install AWS ParallelCluster in a virtual environment \(recommended\)<a name="install-v3-virtual-environment"></a>

We recommend that you install AWS ParallelCluster in a virtual environment to avoid requirement version conflicts with other `pip` packages\.

**Prerequisites**
+ AWS ParallelCluster requires Python 3\.6 or later\. If you don't already have it installed, [download a compatible version](https://www.python.org/downloads/) for your platform at [python\.org](https://www.python.org/)\.

**To install AWS ParallelCluster in a virtual environment**

1. If `virtualenv` is not installed, install `virtualenv` using `pip3`\. If `python3 -m virtualenv help` displays help information, go to step 2\.

------
#### [ Linux, macOS, or Unix ]

   ```
   $ python3 -m pip install --upgrade pip
   $ python3 -m pip install --user --upgrade virtualenv
   ```

   Run `exit` to leave the current terminal window and open a new terminal window to pick up changes to the environment\.

------

1. Create a virtual environment and name it\.

------
#### [ Linux, macOS, or Unix ]

   ```
   $ python3 -m virtualenv ~/apc-ve
   ```

   Alternatively, you can use the `-p` option to specify a specific version of Python\.

   ```
   $ python3  -m virtualenv -p $(which python3) ~/apc-ve
   ```

------

1. <a name="activate-virtual-environment-3"></a>Activate your new virtual environment\.

------
#### [ Linux, macOS, or Unix ]

   ```
   $ source ~/apc-ve/bin/activate
   ```

------

1. Install AWS ParallelCluster into your virtual environment\.

------
#### [ Linux, macOS, or Unix ]

   ```
   (apc-ve)~$ python3 -m pip install --upgrade "aws-parallelcluster"
   ```

------

1. Verify that AWS ParallelCluster is installed correctly\.

------
#### [ Linux, macOS, or Unix ]

   ```
   $ pcluster version
   {
     "version": "3.0.0"
   }
   ```

------

You can use the `deactivate` command to exit the virtual environment\. Each time you start a session, you must [reactivate the environment](#activate-virtual-environment-3)\.

To upgrade to the latest version of AWS ParallelCluster, run the installation command again\.

------
#### [ Linux, macOS, or Unix ]

```
(apc-ve)~$ python3 -m pip install --upgrade "aws-parallelcluster"
```

------