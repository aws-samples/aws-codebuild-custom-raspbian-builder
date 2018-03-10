## AWS CodeBuild Samples
Build custom Raspbian OS images from verified Source with AWS CodeBuild.
* Supports bare-metal & Docker container installs.  
See descriptions below and full details in the [raspbian-docker](./raspbian-docker/README.md) & [raspbian-baremetal](./raspbian-baremetal/README.md) docs.  
* Provisioned via AWS CloudFormation.

## Raspbian Custom Baremetal
AWS project to build a custom Raspbian image from verified source.  

* Outputs custom OS images for a Raspberry Pi to Amazon S3, ready to download, burn to SD card and deploy.  
* Stock customizations provided as Cloudformation parameters include wifi settings, user credentials and an option to install Docker.  
* Further customizations are supported by modifying AWS CodeBuild's buildspec.yml and Dockerfile inline in the AWS CloudFormation template.  
* Per device customizations are supported by the Startup script mechanism documented below.  
* Provisioned as a single CloudFormation template.  

## Raspbian Custom Docker
AWS project to build a custom Raspbian Docker image from verified source.  
* Outputs custom, reduced, Raspbian Docker images to Amazon Elastic Container Registry (ECR).  
* As of 3/2018 there are no [official](https://docs.docker.com/docker-hub/official_repos/) Raspbian Docker images.  
This project builds images from SHA256-verified Raspbian source, so you are assured of a secure chain-of-custody of your base image.  
**Note that ARM binary official images are now available for Debian and Ubuntu.*  
* Enables customization at build time via the included Dockerfile.  
* Installs [Qemu](https://www.qemu.org) in the image, enabling it to run on x86 hosts for test and dev.    
* Provisioned as a single CloudFormation template.  

## License

This library is licensed under the Apache 2.0 License. 
