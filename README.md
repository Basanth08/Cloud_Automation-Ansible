# Cloud Automation with Ansible

## Overview
I automate cloud infrastructure provisioning and management using Ansible. This project demonstrates how I set up secure, highly available AWS environments with Infrastructure as Code (IaC) principles.

---

## Scenario
As part of the AWS Cloud Management Team, I regularly deploy and set up infrastructure on the cloud. For every new project, I use a secure and highly available (HA) VPC and handle regular requests for cloud resources and configurations.

---

## Problem
When I work with VPCs, I deal with many moving parts—subnets, NAT gateways, internet gateways, route tables, network ACLs, and security groups. Manual management is time-consuming and error-prone, which can lead to non-functional or exposed VPCs.

---

## Solution
- I use Ansible for configuration management of VPCs.
- I automate setup to eliminate human errors.
- I centralize change management and use version control (IaC).

---

## Architecture

![Cloud Automation Architecture](architecture.png)

*This diagram shows the automated flow for provisioning AWS resources using Ansible.*

---

## Flow of Execution

1. I logged in to AWS.
2. I created an EC2 instance to run my Ansible playbooks.
3. I installed Ansible and the `boto` library.
4. I set up an EC2 role for Ansible.
5. I created a project directory and sample cloud task (with key pair).
6. I created variable files for VPC and Bastion host.
7. I created playbooks for VPC setup and Bastion setup.
8. I used a `site.yml` playbook to call both at once.

---

## Setup and Installation Log

- I SSHed into my EC2 instance:
  ```bash
  ssh -i ansible-ohio-key.pem ubuntu@18.191.203.32
  ```
- I installed Ansible:
  ```bash
  sudo apt update
  sudo apt install ansible -y
  ```
- I troubleshooted network issues by updating security group outbound rules to allow all outbound traffic.
- I verified Ansible installation:
  ```bash
  ansible --version
  ```
- I installed AWS CLI using the official installer:
  ```bash
  curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
  sudo apt update
  sudo apt install unzip -y
  unzip awscliv2.zip
  sudo ./aws/install
  aws --version
  ```
- I verified my IAM role:
  ```bash
  aws sts get-caller-identity
  ```
  Output:
  ```json
  {
      "UserId": "AROA5FTZB6VPTSKO2G5VT:i-0c32ef1f11a001524",
      "Account": "905418241375",
      "Arn": "arn:aws:sts::905418241375:assumed-role/ansible-admin/i-0c32ef1f11a001524"
  }
  ```

---

## Sample Playbook: Creating and Storing an EC2 Key Pair

Here’s a complete example of how I use Ansible to create an EC2 key pair, print the output, and save the private key to a PEM file:

```yaml
- hosts: localhost
  connection: local
  gather_facts: False
  tasks:
    - name: sample ec2 key
      ec2_key:
        name: sample
        region: us-east-2
      register: keyout

    - debug:
        var: keyout

    - name: store login key
      copy:
        content: "{{ keyout.key.private_key }}"
        dest: ./sample-key.pem
```

**Explanation:**
- I run this playbook locally.
- I use the `ec2_key` module to create a key pair named `sample` in the `us-east-2` region.
- I register the output as `keyout`.
- I print the output using the `debug` module.
- I store the private key in a file called `sample-key.pem` using the `copy` module, so I can use it to log in to my EC2 instances.

---

## VPC Setup Playbook Execution and Output

When I ran my `vpc-setup.yml` playbook, all tasks completed successfully. Here’s what happened:

- All resources were created without errors. Each step in the playbook returned an `ok` status.
- The playbook printed out the IDs for all created subnets, the internet gateway, NAT gateway, and route tables. For example:
  - `pubsub2_out.subnet.id`: `subnet-0b1323e3db4a035dd`
  - `pubsub3_out.subnet.id`: `subnet-0ec67e225226db62b`
  - `privsub1_out.subnet.id`: `subnet-077436b39538fb36d`
  - `privsub2_out.subnet.id`: `subnet-0ce702cf4e2ee5e06`
  - `privsub3_out.subnet.id`: `subnet-0ae587e48b37e439f`
  - `igw_out.gateway_id`: `igw-01bdd784c0d623477`
  - `pubRT_out.route_table.id`: `rtb-082f6cc268d437a24`
  - `NATGW_out.nat_gateway_id`: `nat-00762e9a9ff6cba39`
  - `privRT_out.route_table.id`: `rtb-07c95fe077e6ae6c9`
- The playbook stored all these IDs as Ansible facts and wrote them to a file (`vars/output_vars`) for use in later playbooks (like my bastion host setup).
- The final play recap showed all tasks were successful, with no failures or unreachable hosts:
  ```
  PLAY RECAP *****************************************************************************************************
  localhost                  : ok=15   changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
  ```

**What this means:**
- My VPC, subnets, internet gateway, NAT gateway, and route tables were all created and are ready for use.
- I now have all the resource IDs I need for further automation (e.g., launching EC2 instances, setting up security groups, etc.).
- The environment is reproducible and can be torn down and recreated as needed.

---

## Why Are There So Many Subnets in My AWS Console?

When I look at the AWS console, I see multiple subnets listed. Here’s why:

- The subnets with names like `vprofile_pubsub1`, `vprofile_pubsub2`, `vprofile_pubsub3`, `vprofile_privsub1`, `vprofile_privsub2`, and `vprofile_privsub3` are the ones I created with my Ansible playbook in my custom VPC. Their Subnet IDs match the ones output by my playbook.
- The subnets with no name (just a dash “–”) and different VPC IDs are part of the default VPC that AWS creates in every region. These are not related to my automation and are present in every AWS account by default.

**This is not a problem!**
- Having both default and custom subnets is normal in AWS.
- My automation is working as expected, and the named subnets are the ones I created.
- I can safely ignore the default subnets unless I want to clean up my AWS account or am running into subnet limits.

---

## Project Files Overview

- **README.md**: Project documentation, scenario, setup, troubleshooting, and playbook examples.
- **architecture.png**: Architecture diagram for the automation flow.
- **bastion-instance.yml**: Ansible playbook to set up a bastion host, including key creation, security group, and EC2 instance launch.
- **vpc-setup.yml**: Ansible playbook to set up a VPC, subnets, gateways, and route tables, and output resource IDs.
- **vars/**: Contains variable files for playbooks (`bastion_setup`, `vpc_setup`, and generated `output_vars`).
- **Diagrams/**: (Currently empty)

---

## Project Structure

```
Cloud_Automation-Ansible/
├── playbooks/           # Ansible playbooks
├── roles/               # Ansible roles
├── inventory/           # Inventory files
├── group_vars/          # Group variables
├── host_vars/           # Host variables
├── vars/                # Variable files
├── architecture.png     # Architecture diagram
├── README.md            # Project documentation
```

---

