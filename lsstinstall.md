## Installing the LSST stack for AWS HPC Cluster

Key to running an HPC Cluster - whether AWS-based or otherwise - is the operating software. On all its compute resources, AWS uses Amazon Machine Images (AMIs) to spin-up instances running the desired operating system and other software. Users can build-upon and customise AWS's default AMIs and save them for their own future use. In this respect, AMIs are not too dissimilar to Docker Images.

When using a standalone machine, I prefer to use (slightly) customised versions of the LSST Docker Images provided by the DM team to process data. I've found, however, that running Docker containers on an AWS HPC Cluster makes things unnecessarily complicated, and that creating my own customised AMI with the LSST stack pre-installed is (currently) a simpler and more elegant solution. In this way, all compute nodes spin-up with the LSST stack already installed and ready to use.

In this section, I'll take you through the process of selecting an appropriate base AMI to customise, how to install the LSST stack on this AMI, and finally how to save this customised AMI for future use. If, however, you'd prefer to just use my own customised AMI and skip the rest of this section, then feel free to continue to the next page.
