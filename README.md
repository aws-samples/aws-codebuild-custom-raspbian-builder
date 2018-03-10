## AWS CodeBuild Custom Raspbian Builder
Build custom Raspbian OS images from verified Source with AWS CodeBuild.
* Supports bare-metal & Docker container installs.  
See descriptions below and full details in the [raspbian-docker](./raspbian-docker/README.md) & [raspbian-baremetal](./raspbian-baremetal/README.md) docs.  
* Both projects provisioned as single-step CloudFormation templates.  
Code embedded in the templates is copied to S3 to start the build pipeline without end-user intervention.  
* [Qemu](https://www.qemu.org) emulation is used to build ARM native Raspbian on x86 based CodeBuild.  

## Raspbian Custom Baremetal
AWS project to build a custom Raspbian image from verified source.  
* Outputs custom OS images for a Raspberry Pi to Amazon S3, ready to download, burn to SD card and deploy.  
* Instead of installing, updating and customizing on-device, all the heavy lifting is done in AWS CodeBuild.  
* Secure headless-setup:  SSH is enabled, but the default user is replaced with your own username.  
* Built-in option to set wifi credentials at build time.  
* Built-in option to install Docker.  
* Per device customizations are supported by a startup script mechanism accessed by mounting the /boot partition on your Mac or PC.  
* Further customizations are supported by modifying AWS CodeBuild's buildspec.yml and Dockerfile inline in the AWS CloudFormation template.  
* SD image is automatically reduced to minimize download size.

## Raspbian Custom Docker
AWS project to build a custom Raspbian Docker image from verified source.  
* Outputs custom, reduced, Raspbian Docker images to Amazon Elastic Container Registry (ECR).  
* As of 3/2018 there are no [official](https://docs.docker.com/docker-hub/official_repos/) Raspbian Docker images.  
This project builds images from SHA256-verified Raspbian source, so you are assured of a secure chain-of-custody of your base image.  
**Note that ARM binary official images are now available for Debian and Ubuntu.*  
* Enables customization at build time via the included Dockerfile.  
* Installs [Qemu](https://www.qemu.org) in the image, enabling it to run on x86 hosts for test and dev.     

## License

This library is licensed under the Apache 2.0 License. 
