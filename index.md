## So you want to run the LSST stack on an AWS cluster?

Cloud computing services like Amazon Web Services (AWS) and Google Cloud provide access to almost unlimited computing resources without the need to invest in large amounts of hardware. Using services like AWS's ParallelCluster, multiple cloud-computing nodes can be linked together to behave as a traditional High Performance Computing (HPC) cluster that use multi-processor standards that are familiar to many researchers (e.g., OpenMPI, NFS, Slurm, etc.).

With many aspects of the LSST stack already optimised for multi-processor systems, and having been inspired by a presentation by Hsin-Fang Chiang and Nate Lust at the 2019 LSST Project and Community Workshop, I thought I'd have a go at setting up an AWS cluster and running the LSST stack on it. While eventually successful, the process wasn't entirely straightforward, so I decided to document what I did for future reference and to assist others who also may want to run the LSST stack on an AWS HPC cluster.

### Creating an AWS account

To set up an AWS HPC cluster you will first need an AWS account with the required priveleges. To create an account, go to [https://portal.aws.amazon.com/billing/signup](https://portal.aws.amazon.com/billing/signup) and follow the instructions. Registration requires a credit card - after all, Jeff Bezos has to put food on the table somehow - and unfortunately some of the steps are outside the free tier (meaning your credit card will be charged). However, provided you ensure you delete any clusters you create, then following this tutorial should cost you way less than 5 US$.

When you first register with AWS, it automatically creates a root user account. However, for security reasons it is strongly recommended that you _don't_ use this root account for everyday tasks. Instead, it's recommended that you create extra user accounts that have more restricted access than the root user and use those user accounts to interact with AWS services.

To create a new user, log into AWS as the root user then:

1. Click on **Services** then in the resulting drop-down menu select **IAM** under *Security, Identity, & Compliance*.
2. In the left-hand menu of the IAM page, select **Users**.
3. Then click on the blue **Add User** button in the main window, which starts the *Add User* Wizard.
4. In first page of the *Add User* Wizard, enter a name for the new user (e.g., ClusterUser), and select both *Programmatic access* and *AWS Management Console access* (the latter isn't strictly necessary, but it'll make your life a whole lot easier).
5. Choose whether you want to auto-generate your new user password, and whether you want to have to reset your password the first time you log in as the new user (the latter is more typically used when the root user differs in-person from the new user, but this is unlikely to be the case here). Click on the **Next:Permissions** button.
6. The *Add Permissions* page allows you to give various priveleges to the new user. We'll cover permissions in more detail later. For now, select **Attach existing policies directly** and in the resulting table select **AdministratorAccess** then click on **Next:Tags**.
7. On the *Add Tags* page you can add some tags, which can help to associate users and resources to certain projects. You can't really do any damage here, but unless you really want to, I wouldn't bother adding any tags. Click on **Next:Review**.
8. Make sure everything looks ok on the *Review* page, then click on **Create User**.
9. Once the new user has been created, you'll be presented with a *Success* page. Either make a note of the new login credentials (including the *Access key ID* and *Secret access key*), or, more preferably, download the csv file. You can also choose to send an email to yourself, but this doesn't contain the access keys.


