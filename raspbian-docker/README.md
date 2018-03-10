# Raspbian Custom Docker
AWS project to build a custom Raspbian Docker image from verified source.  

* Outputs custom, reduced, Raspbian Docker images to Amazon Elastic Container Registry (ECR).  
* As of 3/2018 there are no [official](https://docs.docker.com/docker-hub/official_repos/) Raspbian Docker images.  
This project builds images from SHA256-verified Raspbian source, so you are assured of a secure chain-of-custody of your base image.  
**Note that ARM binary official images are now available for Debian and Ubuntu.*  
* Enables customization at build time via the included Dockerfile.  
* Installs [Qemu](https://www.qemu.org) in the image, enabling it to run on x86 hosts for test and dev.    
* Provisioned as a single CloudFormation template.  

![Architecture Diagram](/images/docker_architecture.png)


__Works with:__ 
* Raspbian Jessie

__Instructions:__  
* Create an AWS CloudFormation stack using the provided template.
* All resources are named using the Cloudformation Stack name you choose.  
Because of this, the Stack name must be compatible with the name restrictions of all services deployed by the stack.  (a-z 0-9 _ -)  
* CloudFormation will run a CodePipeline/CodeBuild job that outputs the Raspbian image to Amazon Elastic Container Registry in 10-20 minutes.  
* The resulting image can be found in the ECR repository shown in the Stack Output: *ecrDestinationRepositoryName*.  
* Download the image with Docker by first [authenticating with ECR](https://docs.aws.amazon.com/AmazonECR/latest/userguide/Registries.html#registry_auth) then [pulling the image](https://docs.aws.amazon.com/AmazonECR/latest/userguide/docker-pull-ecr-image.html).     
* For faster build-times, download and copy the Raspbian OS source into a public S3 bucket in the same region.  
Then use the s3 url as the *raspbianSourceUrl* input to CloudFormation.  
* Direct modifications can be made to the OS filesystem when mounted in CodeBuild.  
* Modifications requiring running binaries in Raspbian (apt-get etc.) can be made via the Dockerfile.    
* The resulting image is ~50MB compressed ~115MB uncompressed, matching Debian's official image size.  

__Resources deployed by Cloudformation:__
* S3 Bucket for source code.
* ECR Repository for Raspbian image output
* CodePipeline
* CodeBuild Project
* Lambda Function to zip and copy source code for CodePipeline
* IAM roles and permissions

__Steps in build process:__
* Lambda puts a source build.zip file in the S3 Bucket.  
*build.zip* contains the *Dockerfile*, *builspec.yml* and other files used by CodeBuild.  
The file contents are included inline in the *sourceToS3* section of the Cloudformation Template.   
* CodePipline detects a new s3 put-object on *build.zip* to trigger the CodeBuild Project.  
* CodeBuild gets *build.zip* from s3, runs the build and pushes the image to ECR.  
* Build details are written to */buildinfo.txt* inside the Raspbian image. 

__CodeBuild process details:__  
* [Qemu](https://www.qemu.org) emulation is installed in the CodeBuild environment to allow us to run an ARM binary container on CodeBuild's underlying x86 ec2 instance.  
* Full Raspbian install image is downloaded and SHA256 verified.  
* root & boot partitions from the Raspbian image are mounted into the CodeBuild environment.  
* Directly modify the mounted filesystems.
* Tar the mounted filesystems and import into Docker as a Docker Image.  
* Run a docker build with the included Dockerfile, using the Docker Image built in the previous step as the source.  
* Because we have a running copy of the ARM/Raspbian OS, Docker can execute commands (apt-get etc) to modify the operating system.
* Export the running container to a tar file, then re-import to flatten and reduce the image.  
* Push the image to ECR. 
* Full CodeBuild steps can be found in the *buildspec_body* and *dockerfile_body* sections of the CloudFormation template.  


 
