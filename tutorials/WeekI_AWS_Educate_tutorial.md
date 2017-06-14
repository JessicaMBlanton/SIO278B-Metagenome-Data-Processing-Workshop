# Using AWS EDUCATE

*SIOB278 Ocean 'Omics: Metagenome Data Processing & Analysis workshop tutorial*

* Last edit: 4/15/17
* Jessica Blanton
 
For our workshop, we will be using Amazon Web Services
AWS Educate, provided by UCSD ITS/ETS. This will give us scaled-back access to some services, namely Elastic Cloud Computing (EC2), with a starting budget of $50 per person!

**Tutorial Goals**:

Learn how to correctly start an instance, adjust storage, and set up a working directory.

**Links**:

[**UCSD AWS Educate console sign-in**](https://awsed.ucsd.edu/courses) 

[**Full Spring 2017 Instructions**](https://docs.google.com/document/d/1kuDwk_MyZ0BpgyHorPlUsfqvY3gl_wIJc34DgBFE6cI/edit?usp=sharing)



## Launching an instance for SIOB278
**1. Choose AMI**

>Ubuntu Server 16.04 LTS (HVM), SSD Volume Type - ami-a58d0dc5

**2. Choose Instance Type**

>m4.2xlarge
	
**3. Configure Instance** - Stick to one ZONE for the duration of the class: choose team "a", "b", or "c"

>subnet-108d4059 | Default in us-west-2**a**
	
**4. Add Storage** - increase size: 

>Size (GiB) | 100GB

>Volume Type | General Purpose SSD (GP2)

**5. Add Tags** - required to leave as defaults 

**6. Configure Security Group**

> Assign a security group: Select an existing security group

then

> sg-22c75d5a | Inbound from UCSD Campus Hosts Only

**7. Review** - check over everything...

**8. Hit launch**

**9a. Keypair for First Instance**: Create a keypair - you only need one for the whole course!  Please use the format: SIOB278\_SI17\_USERNAME

**9b. Keypair for any following instances**: Use existing keypair 

**10. Save your key, SIOB278\_SI17\_USERNAME.pem** in a safe place on your local laptop, and **set the permissions via command line**:

	chmod 400 /Users/jess/Documents/Archived_Docs/SIOB278_SI17_jmblanto.pem

**10. ssh into your new machine using your key**:

*You must be using a UCSD host (on campus network, or using VPN)*

	ssh -i /Users/jess/Documents/Archived_Docs/SIO278B_SI17_jmblanto.pem ubuntu@ec2-52-39-197-22.us-west-2.compute.amazonaws.com



## Setting up your instance 

Change time zone to your local time
	
	sudo rm /etc/localtime
	sudo ln -s /usr/share/zoneinfo/America/Los_Angeles /etc/localtime

Update instance software

	sudo apt-get update

list block devices to see what volumes you have available

	lsblk
	
		NAME    MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
		xvda    202:0    0  100G  0 disk 
		└─xvda1 202:1    0  100G  0 part /
	
Make a directory on your [already mounted] root disc

	mkdir /home/ubuntu/raw_reads/

Check out where your discs are :

	df -h
	
		Filesystem      Size  Used Avail Use% Mounted on
		udev             16G     0   16G   0% /dev
		tmpfs           3.2G  8.6M  3.2G   1% /run
		/dev/xvda1       99G 1000M   94G   2% /
		tmpfs            16G     0   16G   0% /dev/shm
		tmpfs           5.0M     0  5.0M   0% /run/lock
		tmpfs            16G     0   16G   0% /sys/fs/cgroup
		tmpfs           3.2G     0  3.2G   0% /run/user/1000

Moving into your newly created directory should now work, and you should be all set!

	cd /home/ubuntu/raw_reads/
	
-
**Note: the tilde “~” is a symbol for “home”, meaning that:

	~/raw_reads/

is the same as

	/home/ubuntu/raw_reads/

Just to save us some extra keystrokes!
	
___
___
___


## Working with additional storage volumes

**Note:
Volumes are either added at the time the instance is launched (default storage), or created independently.
 
**Note:
Regardless of how they came to be they can be attached or detached and moved between different instances, but only when the instances are stopped.

*Mounting additional discs* 
	
	# make directory (for reattached volumes, this will become the name of the root directory)
	sudo mkdir /disk100
	
	# mount on volume
	sudo mount /dev/xvdf /disk100
	
	# change permissions of disc
	sudo chmod a+rwxt /disk100

*To unmount a disc*

	sudo umount /disk100
-
### Charges for running an instance
* Running instances are charge at posted @ list price
* Stopped instances are not charged
* Even if you start and immediately stop an instance, you get charged a FULL hour. No prorating


### Charges for storage (attached + detached EBS volumes)

* EBS volumes are charged for storage, whether or not attached to a running instance
* Cost of EBS volume - is as a provisioned block at $0.10 per GB-month, prorated to the 1h
