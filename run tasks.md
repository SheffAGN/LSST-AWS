## Part 5: Running multi-node LSST tasks on an AWS cluster

After launching and connecting (i.e., ssh'ing) to an AWS cluster, the task of submitting tasks to process your data across multiple nodes should be pretty familiar to those of you with experience of using the LSST stack on a cluster environment. While in the following I will provide some commands to demonstrate how to distribute tasks across multiple nodes, this is by no means a tutorial on how to *run* the stack. For that, I recommend you check out the [LSST Stack Club](https://github.com/LSSTScienceCollaborations/StackClub) tutorials.

### Setting up

If you don't already have an AWS cluster already running, I recommend you launch one now using the following command from Part 4 of this tutorial:
    
    >> pcluster create lsst

and once it has launched, connect to the master node with:

    >> pcluster ssh lsst -i /path/to/keyfile.pem
    
Once you're connected, the first thing you'll need to do prior to running any tasks is to tell the cluster where the LSST stack is installed. To do this, use your favourite editor (I use emacs; if it's not already installed, you can install it using `yum`) to edit the `/home/centos/.bashrc` file to include the following line:

    >> source /mnt/lsst_stack/loadlsst.bash
    
*Note: Here I've assumed that you have mounted the LSST stack volume on `/mnt/lsst_stack`, as used in Part 4. If you've mounted it somewhere else, you'll need to edit the above line accordingly.*

By putting the above line in your `.bashrc` file, you're ensuring that every compute node sources the `loadlsst.bash` file at login, thereby ensuring that every node has access to the LSST stack installed on the mounted volume. This is possible as the `/home` directory is also shared across all nodes.

Next, you'll need to either log out and log back into to the master node, or source your newly-edited `.bashrc` file for the changes to take effect. Once you've done that, you should be able to run the stack's `setup` command to give you access to command line tasks, e.g.:

    >> setup obs_subaru

### Running an LSST command line driver

With the stack setup, you can now issue command line drivers, which are LSST stack tasks that are specifically written make use of multiple nodes (using MPI).
