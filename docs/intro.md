---
sidebar_position: 1
slug: /
---

# About project

This project is a portfolio project designed to demonstrate my skills in infrastructure management, and Infrastructure as Code (IaC) practices. The primary goal of this project is to host a static website using CI/CD pipeline. The project integrates several modern tools and services to achieve an efficient and flexible deployment process.

** The site is stored in this [Github repository](https://github.com/lilPenny/MyWebPro.git) with Gh Action to build web and store dockerfile in dockerhub**

** This [Github repository](https://github.com/lilPenny/MyWebProIAAC) is for IaC**

## Technologies Used

**AWS:** The project is hosted on an EC2 VPS, which serves as the primary compute instance.

**GitHub Actions:** Automating the build and deployment pipeline for continuous integration and delivery.

**Docker:** Containerizing the static website to ensure consistent deployment across different environments.

**Terraform:** Provisioning AWS infrastructure, including EC2 instances, security groups, and other required resources.

**Ansible:** Automating EC2 instance configuration, Docker installation, and application deployment. AWS Systems Manager (SSM) is used to execute the Ansible playbooks from the repository, while Parameter Store securely manages environment variables and sensitive configuration data.

**Traefik:** Acting as the reverse proxy routing traffic to the Docker container, integrated with Cloudflare DNS for domain management. Traefik also works with Let's Encrypt to automatically generate and renew SSL certificates, ensuring secure HTTPS access to the website.

## GitHub Actions

### Docker Build and Push Workflow
This workflow automates the building and pushing of a Docker image to Docker Hub. It performs tasks like logging into Docker, setting up build tools (QEMU, Buildx), and then building and pushing the Docker image with the latest version tag.

### Terraform Infrastructure Workflow
This workflow handles the provisioning and management of AWS infrastructure using Terraform. It can perform actions like plan, apply, or destroy. It ensures the infrastructure is properly initialized, validated, formatted, and executed based on the action triggered.

### Ansible SSM Workflow 
This workflow is responsible for running Ansible playbooks on the EC2 instances via AWS Systems Manager (SSM). Creates an SSM association using AWS CLI, linking the Ansible playbook to the EC2 instances based on tag via the SSM service. The playbook is automatically executed on the instances using AWS-ApplyAnsiblePlaybooks.

## Terraform
This Terraform configuration automates the creation and management of AWS infrastructure necessary for hosting the web application.

**EC2 Instance Creation:**
It provisions an EC2 instance that will host the application. The instance is configured with a specific AMI and instance type suitable for running the Dockerized web app.

**Security Groups:**
Terraform defines security groups to control access to the EC2 instance:

SSH access is restricted to your IP address for secure administration.

HTTP (port 80) and HTTPS (port 443) access are opened for Cloudflare’s IP ranges, as Cloudflare is used as the reverse proxy and CDN for the website.

**IAM Role for SSM:**
An IAM role is created for the EC2 instance that allows it to interact with AWS Systems Manager (SSM) for configuration and management tasks. This role provides permissions for the EC2 instance to execute commands via SSM.

**Key Pair:**
Terraform generates an SSH key pair, which is used for securely accessing the EC2 instance. The public key is associated with the EC2 instance upon launch.

**S3 for Terraform State Storage:**
To manage Terraform state files securely and to enable collaboration, the Terraform state is stored in an S3 bucket. This ensures that the state of the infrastructure is tracked and available for future updates, with versioning enabled for consistency.

## Ansible
Ansible is used to automate the configuration and management of the EC2 instance after it has been provisioned by Terraform. The key tasks handled by Ansible are:

**EC2 Instance Configuration:**
Ansible ensures the EC2 instance is properly set up for hosting the Dockerized application. This includes installing necessary dependencies, setting up Docker, and running composed traefik container.

**Script and Cron Job Setup:**
In the initial stage of the deployment, Ansible is used to configure a script and a cron job on the EC2 instance. The script, which is responsible for updating the Cloudflare DNS record, is automatically executed after the instance restarts. A cron job is created in the system’s crontab using the @reboot directive, which triggers the script to run after the system reboots

**SSM Integration:**
Ansible utilizes AWS Systems Manager (SSM) to manage the EC2 instance remotely. This allows the Ansible playbook to be executed directly from the repository via SSM without needing SSH access to the EC2 instance. It simplifies management and provides better security by not requiring direct SSH keys for access.

## Traefik Setup
Traefik is used as a reverse proxy for the application routing incoming HTTP and HTTPS traffic to the containers and redirects http to https. It also automatically handles SSL certificate generation using Let's Encrypt and Cloudflare DNS API.

**Cloudflare integration:** Traefik uses Cloudflare's DNS API to handle SSL certificates through DNS-01 challenges, which ensures that the site is securely served over HTTPS.

**Container configuration:** The myWeb container is linked with Traefik by labels. These labels define how Traefik should route traffic:

The HTTP router listens on the web entrypoint and redirects to HTTPS.

The HTTPS router ensures secure communication and uses the cloudflare certificate resolver for automatic SSL certificate provisioning via Let's Encrypt.

**Traefik Configuration:**
EntryPoints: Traefik is configured to listen on two entry points:

web: Handles HTTP traffic, redirecting it to HTTPS.

websecure: Handles HTTPS traffic.

**CertificatesResolvers:** The configuration uses the Cloudflare DNS challenge to automatically generate SSL certificates with Let's Encrypt.

This setup ensures that Traefik automatically handles the routing of traffic to the web application and secures it using Let's Encrypt, all while utilizing Cloudflare's DNS API for DNS challenge-based certificate generation.