# Raspbian Custom Baremetal
AWS project to build a custom Raspbian image from verified source.  

* Outputs custom OS images for a Raspberry Pi to Amazon S3, ready to download, burn to SD card and deploy.  
* Stock customizations provided as Cloudformation parameters include wifi settings, user credentials and an option to install Docker.  
* Further customizations are supported by modifying AWS CodeBuild's buildspec.yml and Dockerfile inline in the AWS CloudFormation template.  
* Per device customizations are supported by the Startup script mechanism documented below.  
* Provisioned as a single CloudFormation template.  

![Architecture Diagram](/images/baremetal_architecture.png)


__Tested on:__ 
* Raspberry Pi 3B, 2B, 1B+ & Zero W with Raspbian Jessie & Stretch

__Instructions:__  
* Create an AWS CloudFormation stack using the provided template.
* All resources are named using the Cloudformation Stack name you choose.  
Because of this, the Stack name must be compatible with the name restrictions of all services deployed by the stack.  (a-z 0-9 _ -)  
* CloudFormation will run a CodePipeline/CodeBuild job that outputs the Raspbian image to Amazon S3 in 10-20 minutes.  
* The resulting image can be found in the S3 Bucket shown in the Stack Output: codebuildSourceBucket.  
* Download the image, burn to SD, and boot your Raspberry!  
*We have had great success using Resin.io's [Etcher](https://etcher.io) to burn the image to SD cards.*  
* Direct modifications can be made to the OS filesystem when mounted in CodeBuild.  
* Modifications requiring running binaries in Raspbian (apt-get etc.) can be made via the Dockerfile.  
* For per-device cusomization, see the *Startup script mechanism* section below.  
* For faster build-times, download and copy the Raspbian OS source into a public S3 bucket in the same region.  
Then use the s3 url as the *raspbianSourceUrl* input to CloudFormation.

__Resources deployed by Cloudformation:__
* S3 Bucket for source code and Raspbian image output.
* CodePipeline
* CodeBuild Project
* Lambda Function to zip and copy source code for CodePipeline
* IAM roles and permissions

__Steps in build process:__
* Lambda puts a source build.zip file in the S3 Bucket.  
*build.zip* contains the *Dockerfile*, *builspec.yml* and other files used by CodeBuild.  
The file contents are included inline in the *sourceToS3* section of the Cloudformation Template.   
* CodePipline detects a new s3 put-object on *build.zip* to trigger the CodeBuild Project.  
* CodeBuild gets *build.zip* from s3, runs the build and pushes the image to S3.  
* Build details are written to */boot/buildinfo/buildinfo.txt* inside the Raspbian image. 

__CodeBuild process details:__  
* [Qemu](https://www.qemu.org) emulation is installed in the CodeBuild environment to allow us to run an ARM binary container on CodeBuild's underlying x86 ec2 instance.  
* Full Raspbian install image is downloaded and SHA256 verified.  
* root & boot partitions from the Raspbian image are mounted into the CodeBuild environment.  
* Directly modify the mounted filesystems.
* Tar the mounted filesystems and import into Docker as a Docker Image.  
* Run a docker build with the Dockerfile, using the Docker Image built in the previous step as the source.  
* Because we have a running copy of the ARM/Raspbian OS, Docker can execute commands (apt-get etc) to modify the operating system.
* Export the running container to a tar file, and unpack into CodeBuild's working directory.  
* Create the /boot/startup-scripts and first_run_setup.sh system (see section below) in the unpacked OS directory.  
* Build a new Raspbian install image from the OS directory.  
The image will be appropriately sized based on the storage needed by the modified OS.  
* Push the new image to S3.  
* Full CodeBuild steps can be found in the *buildspec_body* and *dockerfile_body* sections of the CloudFormation template.  

__Startup script mechanism:__
* The Raspbian install image contains an ext4 root partition and a FAT32 /boot partition.  
Mounting ext4 on OSX and Windows is an involved process, while mounting FAT partitions is natively supported.  
To enable per-device customization from a single image, and ongoing customizations of the SD card from an OSX or Win host, we created a startup script system accessible from the mounted /boot partition.  
* Scripts in /boot/startup-scripts/run-once/ & /boot/startup-scripts/every-time/ are called by /etc/rc.local on start.  
* Files in run-once are then moved to /boot/startup-scripts/ran-once/ to prevent re-running them on next boot.  
* By default, wifi and user credentials are injected into a first_run_setup.sh script in /boot/startup-scripts/run-once/  
* You can modify that script in the built image or SD card to customize your Raspberries individually.  
* Possible uses include custom hostnames, static IP addresses or AWS IoT x509 certificates.  
 
