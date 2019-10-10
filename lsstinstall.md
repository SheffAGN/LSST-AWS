## Installing the LSST stack for use on an AWS HPC Cluster

Key to running an HPC Cluster - whether AWS-based or otherwise - is the operating software. On all its compute resources, AWS uses Amazon Machine Images (AMIs) to launch instances running the desired operating system and other software. Users can build-upon and customise AWS's default AMIs and save them for their own future use. In this respect, AMIs are not too dissimilar to Docker Images.

When using a standalone machine, I prefer to use (slightly) customised versions of the LSST Docker Images provided by the DM team to process data. I've found, however, that running Docker containers on an AWS HPC Cluster makes things unnecessarily complicated, and instead install the stack on a separate volume that I mount as a shared drive.

In this section, I'll take you through the process of launching an instance with an appropriate base AMI and installing the stack on an attached volume that we'll then save to use with a cluster.

### Launching an AWS compute instance

To install the stack on an attached volume, you'll first need to launch a compute instance with a base AMI. AWS has specific AMIs for running on their parallel clusters, so we'll need to use one of those as our base AMI. However, we don't have to run these on an AWS cluster for the install step. Instead, we can just launch a standard single AWS compute instance and use that for the installation, thereby saving on costs.

First, you'll need to identify a suitable AMI as a base. A list of AWS PCluster-compatible AMIs can be found on the aws-parallelcluster github page: [https://github.com/aws/aws-parallelcluster/blob/develop/amis.txt](https://github.com/aws/aws-parallelcluster/blob/develop/amis.txt). I suggest selecting one of the CentOS7 AMIs based in your nearest AWS region. For example, I prefer to use ami-0697036b2287f0afc, since it is the CentOS7 AMI for the eu-west-2 (London) region.

Once you've selected an appropriate AMI, it's time to launch an instance:

1. Log in to your AWS account using the new user you created earlier in the tutorial.

2. Click on **Services** (toward the left of the top navigation bar), then in the dop-down menu select **EC2** under *Compute*.

3. Click on the blue **Launch Instance** button in the central panel, which takes you to the *Launch Instance Wizard*.

4. In the first page of the *Launch Instance Wizard*, copy-and-paste your preferred AMI name (in my case, ami-0697036b2287f0afc), hit enter to search, then select **Community AMIs** in the left-hand menu. There should be a single AMI that matches your search - your preferred AMI. Click on the blue **Select** button associated with your AMI.

5. Next, you will be presented with a selection of compute instances for you to choose from. If you're still within your free usage tier, then *t2.micro* will be labelled as being free. Unfortunately, however, *t2.micro* doesn't come with sufficient memory to install the LSST stack. Instead, I recommend using *t2.large* (which, at the time of writing, comes with 8 GB memory). Don't worry too much about the cost; as of 2019, a *t2.large* instance costs 10 US cents per hour, and you'll need to run it for less than an hour. So select the *t2.large* instance type and then click on the grey **Next: Configure Instance Details** button.

6. Unless you know what you're doing, I wouldn't both changing anything on the *Configure Instance Details* page, so click on the grey **Next: Add Storage** button.

7. We're going to install the lsst stack onto a separate volume from that of the operating system. We'll also use separate volume for our reference catalogue. For the stack, add a 15 GB *gp2* volume. For the reference catalog, add a 120 GB *standard* volume (the catalogue is actually only 49 GB, but it's gzipped compressed and tarred, so needs more space while unzipping and untarring; if need be, you can copy the catalogue to a smaller volume once unzipped and untarred). You want the volumes to persist after termination of the instance, so *don't* select *Delete on Termination*. After adding these two volumes click on the **Next: Add Tags** button and, unless you want to add some tags, **Next: Configure Security Group**.

8. AWS uses ssh to allow you to log into running instances. For added security, you can limit the IP addresses from which you can access the instance. If you wish to restrict access to allow only connections from your current IP, then feel free to create a New Security Group, then select **My IP** from the drop-down menu in the *Source* column. Click on the blue **Review and Launch** button.

9. Check that everything looks ok on the *Review Instance Launch* page, then click on the blue **Launch** button.

10. You'll be presented with a windown asking you to *Select an existing key pair or create a new one*. AWS uses ssh key pairs rather than passwords for verification. Select **Create a new key pair** from the drop-down menu, give it a name, select **Download Key Pair**, then save the resulting downloaded file somewhere safe on your local machine. Click on **Launch Instances**.

11. You'll be presented with a confirmation screen, from where you can click on the blue **View Instances**, which will take you to the main instance control panel, where you can see all your running instances.

### Logging into your newly-launched instance and installing the LSST stack

With your instance now running, you now need to ssh into it so that you can install the LSST stack onto the newly-created volume:

1. In the main panel of the instance control panel, there should now be a row showing that your instance is running (or at least setting up). Once it has stopped initializing, select it by clicking on the blue check box to the left and clicking on the **Connect** button to the right of the blue "Launch Instance" button. This will open a popup window contining instructions that your should follow to connect to your running instance.

*Note: The example in the popup window will likely suggest that you connect as root@ec2-x-x-... . However, if you use the CentOS AMI, you will need to instead connect as centos@ec2-x-x-... (i.e., just replace "root" with "centos").*

2. One you have ssh'd into your instance, you'll first need to format and mount the two volumes that you created and attached to your instance at launch time. At the command line of your instance submit the following commands:

First, get the name of your attached volumes (in this case `/dev/xvdf` and `/dev/xvdg`):
    
    >> lsblk

Next, format those volumes:

    >> sudo mkfs -t xfs /dev/xvdf
    >> sudo mkfd -t xfs /dev/xvdg
    
Create mount points:

    >> sudo mkdir /mnt/lsst_stack
    >> sudo mkdir /mnt/ps1

Mount the volumes and ensure that you can write to them as user `centos`:
    
    >> sudo mount /dev/xvdf /mnt/lsst_stack
    >> sudo mount /dev/xvdf /mnt/ps1
    >> sudo chown /mnt/lsst_stack
    >> sudo chgrp /mnt/lsst_stack
    >> sudo chown /mnt/ps1
    >> sudo chgrp /mnt/ps1

3. Next, prepare your instance for the installation of the LSST stack:

Update yum packages:

    >> sudo yum update

And install the most recent version of git:

    >> sudo yum remove git*
    >> sudo yum -y install https://centos7.iuscommunity.org/ius-release.rpm
    >> sudo yum -y install git2u-all

5. To install the LSST stack, change directory (`cd`) to `/mnt/lsst_stack`, then follow the instructions for "Installing using newinstall.sh" on the LSST installation webpages: [https://pipelines.lsst.io/install/newinstall.html](https://pipelines.lsst.io/install/newinstall.html), including the separate section regarding prerequisites. Note that you don't have to do step 2 (`mkdir lsst_stack`) since you've already done this step when mounting the volume.
    
6. As well as installing the LSST stack, you'll also need to download, unzip and untar the reference catalogue.

7. With the LSST stack installed and the reference catalogue unpacked you no longer need this instance. So, to save costs, select the running instance on the instance control panel and click on **Actions > Instance State > Terminate** to kill the instance. The mounted volumes will detach, but will persist. Indeed, you should check that you can see them in the *Volumes* section of the *EC2 Dashboard*.

