# EK_AWS_RNASeq
### Having completed my postdoctoral fellowship and no longer having access to the University of Pittsburgh HTC resources I am building my own cloud compute AWS configuration to continue RNA-Seq bioinformatic analyses Building a cloud compute AWS RNA-Seq instance
### This guide is ment to help researchers without CS background and/or AWS training to start their own bioinformatics cloud setup and for my personal records to return to.
### I had prior setup an AWS Free-tier account following the [getting started with AWS documentation] (https://docs.aws.amazon.com/)
### All local commands run from M1 Macbook Pro running macOS Sonoma within terminal

## AWS command line interface (CLI) setup: following [the AWS CLI guide](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-quickstart.html)
1. Download CLI
	```Bash
	% curl "https://awscli.amazonaws.com/AWSCLIV2.pkg" -o "AWSCLIV2.pkg"```
2. install
	```Bash
	% sudo installer -pkg ./AWSCLIV2.pkg -target /```
3. Test install location
   ```bash
   % which aws
   /usr/local/bin/aws```
4. Test version
	```Bash
	% aws --version
	aws-cli/2.17.25 Python/3.11.9 Darwin/23.6.0 exe/x86_64```
5. Remove install .pkg
	```Bash
	% rm AWSCLIV2.pkg ```
	
## Configure CLI SSO (single sign on); note to change username to your user default and d-XXXXXXXXXX obtained from AWS account IAM Identity Center
1. Configure CLI SSO; enter config parameters as requested inline
	```Bash
	% aws configure sso
	SSO session name (Recommended): username-1
	SSO start URL [None]: https://d-XXXXXXXXXX.awsapps.com/start
	SSO region [None]: us-east-2
	SSO registration scopes [sso:account:access]: sso:account:access
	Attempting to automatically open the SSO authorization page in your default browser.
	If the browser does not open or you wish to use a different device to authorize this request, open the following URL:

	https://device.sso.us-east-2.amazonaws.com/

	Then enter the code:

	TWKW-ZQLF
	The only AWS account available to you is: XXXXXXXXXXXX
	Using the account ID XXXXXXXXXXXX
	The only role available to you is: AdministratorAccess
	Using the role name "AdministratorAccess"
	CLI default client Region [None]: us-east-2
	CLI default output format [None]: json
	CLI profile name [AdministratorAccess-XXXXXXXXXXXX]: username-1
```
2. Adjust CLI SSO if needed; I accidentally entered incorrect output paremeter initially
	```Bash
	% aws configure set output json
	% aws configure get output
	json```

## Setup EC2 Instance on AWS Console by following the [AWS EC2 setup guide](https://aws.amazon.com/ec2/getting-started/).
1.  Initially I am using the free-tier t2.micro AWS linux instance although it will be underpowered for doing any actual bioinformatics work
2.	Once setup Note and copy the Public IPv4 DNS address ec2-XX-XX-XX-XX.compute-1.amazonaws.com  and instance id 
3.	On the EC2 Dashboard left panel, under Network & Security, Key Pairs, Create key pair and auto-download (can only do once or remake!)
4.	Move .pem key to default ssh key location if not downloaded there; change key name as desired
```Bash
% mv T2micro.pem ~/.ssh
```
5. Set Permission on .pem file 
  ```Bash
  % chmod 400 ~/.ssh/T2micro.pem
  ```
## Setup EC2 working environment
1. Login to SSO and authenticate as necessary
```BASH
aws sso login --profile erkoppes-1
```
2. Connect to EC2 t2micro EC2 instance with ssh
```
 Bash
	ssh -i ~/.ssh/T2micro.pem ec2-user@ec2-XX-XXX-XX-XXX.us-east-2.compute.amazonaws.com
```
4. Install Dev tools, with or without verbose output
```Bash
	$ sudo yum groupinstall "Development Tools" -y
	$ sudo yum --quiet --showduplicates groupinstall "Development Tools" -y

5. Download and install Miniconda3
```Bash
	$ wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
	$ bash Miniconda3-latest-Linux-x86_64.sh
	$ echo 'eval "$(/home/ec2-user/miniconda3/bin/conda shell.bash hook)"' >> ~/.bashrc
	$ source ~/.bashrc
	$ conda --version
	conda 24.5.0
	```
	
6. Setup an RNA-Seq Conda environment
```Bash
	$ conda create -n rnaseq python=3.8 
	$ conda activate rnaseq
	```
7. Install conda modules
```Bash
	$ conda install -c bioconda fastqc trimmomatic star salmon
	$ conda install -c conda-forge r-base
	$ conda install -c bioconda bioconductor-deseq2 bioconductor-edger	

8. Add additional modules with pip
```Bash
	$ pip install pyaml-env
	$ pip install multiqc
	$ pip install numpy pandas matplotlib seaborn
	```
9. Check disc space
```Bash
	$ df -h
	(rnaseq) [ec2-user@ip-172-31-5-86 rnaseq]$ df -h                                                           
	Filesystem      Size  Used Avail Use% Mounted on                                                           
	devtmpfs        4.0M     0  4.0M   0% /dev                                                                 
	tmpfs           475M     0  475M   0% /dev/shm                                                             
	tmpfs           190M  440K  190M   1% /run                                                                 
	/dev/xvda1       16G  5.8G   11G  37% /                                                                    
	tmpfs           475M     0  475M   0% /tmp                                                                 
	/dev/xvda128     10M  1.3M  8.7M  13% /boot/efi                                                            
	tmpfs            95M     0   95M   0% /run/user/1000 
```
/dev/xvda1 (Root Volume): ## main file system only 16GiB total 37% used; for most applications
/dev/xvda128: ## for EFI boot files
devtmpfs and tmpfs Filesystems: ## temporary RAM usage
'/' is root for the EC2 instance
this will certainly not be enough storage or memory for full RNA-Seq datasets

## setup S3 and EFS storage to link data
## setup AWS Batch jobs, RNA-Seq Docker image
