## Part 4: Creating an AWS Cluster to run the LSST stack

To recap, you've now created an AWS account and have installed the LSST stack on a AWS volume. We're now going to launch an AWS cluster consisting of one master node and two compute nodes, with each of the latter able to access the installed software to process astronomical images in parallel.

In Part 3 of this tutorial, you used the *AWS control panel* to launch an instance, which you then ssh'd into to install the LSST stack and unpack the reference catalogue. By contrast, an AWS Cluster is launched via the command line using the **pcluster** command that you installed by following the instructions linked to in Part 2 of this tutorial.

Clearly, you'll want to be able to configure your cluster such that it has, for example, the desired number of nodes consisting of the desired instance type, attaches to the volumes you have created, etc. This is achieved with a config file which **pcluster** expects to be located at `~/.parallelcluster/config`; the `.parallelcluster` directory was created when you configured pcluster as part of the installation process. The AWS Parallel Cluster guide includes descriptions of all the parameters that can be set within the config file. You may, however, find it easier to use the following example config file as a starting point:

    [global]
    cluster_template = default
    update_check = true
    sanity_check = true
    
    [aws]
    aws_region_name = eu-west-2
    
    [cluster default]
    base_os = centos7

    cluster_type = ondemand

    master_instance_type = t2.micro
    compute_instance_type = t2.xlarge
    
    initial_queue_size = 2
    max_queue_size = 2
    maintain_initial_size = true
    scheduler = slurm

    custom_ami = ami-xxxxxxxxxxxxxxxxx

    vpc_settings = public
    key_name = your_key_name

    master_root_volume_size = 17
    compute_root_volume_size = 17
    ebs_settings = lsst_stack, data, ps1

    [vpc public]
    vpc_id = vpc-xxxxxxxx
    master_subnet_id = subnet-xxxxxxxx

    [ebs lsst_stack]
    shared_dir = /mnt/lsst_stack
    ebs_volume_id = vol-xxxxxxxxxxxxxxxxx

    [ebs data]
    shared_dir = /mnt/data
    ebs_volume_id = vol-xxxxxxxxxxxxxxxxx

    [ebs ps1]
    shared_dir = /mnt/ps1
    ebs_volume_id = vol-xxxxxxxxxxxxxxxxx

The config file consists of a number of sections, each of which is demarcated with a section name in square brackets: 
- The `[global]` section contains some general settings and also tells pcluster which cluster config to use (in this case, `default`), I suggest you keep these as they are for now.
- The `[aws]` section tells pcluster which AWS region to launch the pcluster in. This *must* be the same region where you created the volumes in Part 3 of this tutorial.
- The `[cluster default]` section contains many of the parameters that define the properties of the cluster. Here, the `default` is that referred to in the `[global]` section.
  - `base_os` tells pcluster what to use as the base operating system. Apparently this is needed, despite the ami also defining the operating system. I don't know what would happen if you used a different base OS from the ami, presumably it wouldn't be good.
  - `cluster_type` can be set to either `spot` for request spot pricing, or `ondemand` for on demand pricing. Spot pricing basically means you pay a lower price for your node-hours, but your job could be thrown off if general demand for instances (by other users) increases. I suggest you use `ondemand` for testing purposes.
  - `master_instance_type` and `compute_instance_type` refer to the instance types to be used for the master node and compute nodes. Since the master node won't be doing any heavy lifting, this can be fairly lightweight. For the compute nodes, you'll need an instance that comes with a substantial amount of RAM; for my data, I needed at least an *t3.xlarge*, but your needs may differ from mine.
  - `initial_queue_size` and `max_queue_size` specify the initial and maximum number of nodes, respectively, that your cluster will consist of.
  - If `maintain_initial_size` is set to `true`, then the number of compute nodes will never drop below the number of nodes specified as `initial_queue_size`. If set to `false`, then the number of compute nodes will scale down to zero if there are no running or queued jobs.
  - `scheduler` specifies the type of batch scheduler to use. Options include `sge`, `torque`, `slurm`, and `awsbatch`. The LSST stack will, I believe, work with a number of different schedulers, but I prefer to use `slurm`.
  - `custom_ami` tells pcluster which ami to use. This MUST be a pcluster AMI specific to your AWS region and base OS (see Part 3 of this tutorial for more details on selecting an appropriate AMI).
  - `vpc_settings` describe which Virtual Private Cloud settings to use. Here, it refers to the `[vpc public]` section below.
  - `key_name` refers to the ssh key that you will use to connect to the master node. Feel free to use the key you generated in Part 3 of this tutorial.
  - `master_root_volume_size` and `compute_root_volume_size` describe the size, in GB, of the HDD volumes connected to each node. This must be large enough to accommodate the ami, but is *not* the volume containing either the LSST stack or the reference catalogue.
  - `ebs_settings` refer to the ebs volume settings specified later in the file; in this case `[ebs lsst_stack]`, `[ebs ps1]`, `[ebs data]`. It is the former two ebs volumes that contain the lsst stack and reference catalogues. These volumes are shared among all the master and compute nodes.
- The `[vpc public]` section specifies the identity of the Virtual Private Cloud (VPC) and Subnet in which the cluster will be launched. Here `public` refers to the value given in the `vpc_settings` parameter within the `[cluster default]` section:
  - `vpc_id` is the ID number of the VPC where your cluster will launch and run. When you set up an AWS account, your are by default given a VPC in every AWS region. Here, you must specify the ID of a VPC within the region in which the volumes you created in Part 3 reside. To get this VPC ID, log into your AWS account, making sure you're in the same region as the aforementioned volumes, click on **Services > EC2** and the default VPC ID (`vpc-********`) will be shown in the rightmost panel of your screen.
  - `subnet-id` is the ID of the subnet where your cluster will launch. Again, this must be in the same availability zone as the volumes created in Part 3. To check which availability zone your volumes are in, log into your AWS account, again making sure you're in the same region as the aforementioned volumes, click on **Services > EC2**, then on the left-hand panel select **Volumes**, and make a record of the value in the *Availability Zone* column in the main panel. Next, click **Services > VPC**, and on the resulting page click **Subnets**, and from the resultint table make a record of the `subnet-id` corresponding to the same *Availability Zone* as your volumes. This is the `subnet-id` you want to use in this parameter.
- The `[ebs *****]` sections include information about the volumes you would like your cluster to connect to. Here, I have three such sections corresponding to the three volumes I'm going to use: one on which the LSST stack is installed; one containing the reference catalogue; and one including my data.
  - `shared_dir` specifies where the volume will be mounted. As the parameter suggests, this mount point will be accessible (i.e., shared) between all master and compute nodes (i.e., all nodes will be able to read/write these volumes).
  - `ebs_volume_id` specifies the ID number of the volume you wish to mount, which you can find by logging into AWS, selecting **Services > EC2**, then clicking on **Volumes** in the left-hand panel. The volume IDs are in the table in the resulting main panel.

Once you have created and saved your config file in `~/.parallelcluster/config`, you are ready to launch your first AWS ParallelCluster. Each AWS Parallel Cluster must be named for identification purposes. In the following, I've simply called it `lsst`. To launch the `lsst` cluster with the above configuration, issue the following command:

    >> pcluster create lsst
    
 It'll take a few minutes for the cluster to be created, but once you are returned to the prompt, you can issue the following command to ssh into the master node of your newly-created cluster:
 
    >> pcluster ssh lsst -i /path/to/keyfile.pem
    
where the `keyfile.pem` is the ssh key file referred to by the `key_name` parameter.

Once you have logged into your cluster, you can start issuing commands to utilise the cluster nodes. Before doing so, however, it's important that you know how to tear-down a cluster to halt charges. To do this, log out of your cluster (in the usual way by just issuing the `exit` command at the command line of the master node) and issuing the following command on your own machine:

    >> pcluster stop lsst
    
Next, we'll look at how to issue LSST stack commands to run across multiple nodes your newly-created AWS cluster.
