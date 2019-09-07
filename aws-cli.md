## The AWS Command Line Interface (CLI) and pcluster

As you witnessed when making a new user, AWS has a pretty nice web interface that allows you to access a wide range of AWS services via point-and-click. However, setting-up and running an AWS HPC Cluster is not currently a services that is supported on the web interface. Instead, it is controlled via the command-line using the **pcluster** command, which you will need to install on your own machine. Thankfully, however, we're all proper scientific researchers here, so we all know-and-love the command line.

# Installing the AWS CLI

Prior to installing pcluster, it's probably wise to install the AWS Command Line Interface (CLI). It's possible that pcluster will work without first installing the AWS CLI, but I've never tried, so I don't know for certain (I'll be interested if anyone out there *does* try to use pcluster without first installing the AWS CLI).

As with all software, how you install the AWS CLI will depend on your operating system, so I will simply refer you to the AWS's own AWS CLI installation pages: [https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html)

# Configuring the AWS CLI

After installing the AWS CLI, you'll need to configure it so that it can, for want of a better term, log-in to your AWS account. To do this, you'll need the access keys that were generated when you created the new user (I hope you made a note of them or saved the csv file). You'll also need to decide which region you want to run your cluster in; I use eu-west-2 (London) since, foolishly, AWS have yet to set up a Yorkshire centre. Again, AWS provide some comprehensive instructions for configuring the AWS CLI, so I'll again [refer you to those](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html).
