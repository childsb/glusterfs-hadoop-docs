To use the automated installer, select one machine in your cluster to run the installer from.  On that machine:

* Open a terminal window and run
   `yum -y install git`
   `git clone https://github.com/jeffvance/glusterfs-cluster-install.git`

* Navigate to the glusterfs-cluster-install directory 

* Create your hosts file per the instructions in the README.txt

* Setup passwordless SSH by running:
   `devutils/passwordless-ssh.sh`

* Launch the installer by running:
   `./install.sh /dev/sdb`

Once this completes you will have gluster installed on your cluster with a gluster volume called HadoopVol mounted on all your nodes.