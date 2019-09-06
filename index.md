## So you want to run the LSST stack on an AWS cluster?

Cloud computing services like Amazon Web Services (AWS) and Google Cloud provide access to almost unlimited computing resources without the need to invest in large amounts of hardware. Using services like AWS's ParallelCluster, multiple cloud-computing nodes can be linked together to behave as a traditional High Performance Computing (HPC) cluster that use multi-processor standards that are familiar to many researchers (e.g., OpenMPI, NFS, Slurm, etc.).

With many aspects of the LSST stack already optimised for multi-processor systems, and having been inspired by a presentation by Hsin-Fang Chiang and Nate Lust at the 2019 LSST Project and Community Workshop, I thought I'd have a go at setting up an AWS cluster and running the LSST stack on it. While eventually successful, the process wasn't entirely straightforward, so I decided to document what I did for future reference and to assist others who also may want to run the LSST stack on an AWS HPC cluster.

### Creating an AWS account

To set up an AWS HPC cluster you will first need an AWS account with the required priveleges. To create an account, go to [https://portal.aws.amazon.com/billing/signup](https://portal.aws.amazon.com/billing/signup) and follow the instructions. Registration requires a credit card - after all, Jeff Bezos has to put food on the table somehow - and unfortunately some of the steps are outside the free tier (meaning your credit card will be charged). However, provided you ensure you delete any clusters you create, then following this tutorial should cost you way less than 5 US$.

When you first register with AWS, it automatically creates a root user account. However, for security reasons it is strongly recommended that you _don't_ use this root account for everyday tasks. Instead, it's recommended that you create extra user accounts that have more restricted access than the root user and use those user accounts to interact with AWS services.

