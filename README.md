# CIS OCI Landing Zone Quickstart Template

## Overview
This Landing Zone template deploys a standardized environment in an Oracle Cloud Infrastructure (OCI) tenancy that helps organizations with workloads needing to comply with the CIS Oracle Cloud Foundations Benchmark v1.1.    

The Landing Zone template deploys a standard three-tier web architecture using a single VCN with multiple compartments to segregate access to various resources. The template configures the OCI tenancy to meet CIS OCI Foundations Benchmark settings related to:

- IAM (Identity & Access Management)
- Networking
- Keys
- Cloud Guard
- Logging
- Events
- Notifications
- Object Storage

 ## Architecture 
 The template creates a three-tier web architecture in a single VCN. The three tiers are divided into:
 
 - One public subnet for load balancers and bastion servers;
 - Two private subnets: one for the application tier and one for the database tier.
 
 The Landing Zone template also creates four compartments in the tenancy:
 
 - A network compartment: for all networking resources.
 - A security compartment: for all logging, key management, and notifications resources. 
 - An application development compartment: for application development related services, including compute, storage, functions, streams, Kubernetes, API Gateway, etc. 
 - A database compartment: for all database resources. 

The architecture diagram below does not show the database compartment, because no resources are initially provisioned into that compartment.
The greyed out icons in the AppDev compartment indicate services not provisioned by this template.

The resources are provisioned using a single user account with broad tenancy administration privileges.

![Architecture](images/Architecture.png)

## How the Code is Organized 
The code consists of a single Terraform root module configuration defined within the *config* folder along with a few children modules within the *modules* folder.

Within the config folder, the Terraform files are named after the use cases they implement as described in CIS OCI Security Foundation Benchmark document. For instance, iam_1.1.tf implements use case 1.1 in the IAM sectiom, while mon_3.5.tf implements use case 3.5 in the Monitoring section. .tf files with no numbering scheme are either Terraform suggested names for Terraform constructs (provider.tf, variables.tf, locals.tf, outputs.tf) or use cases supporting files (iam_compartments.tf, net_vcn.tf).

**Note**: The code has been written and tested with Terraform version 0.13.5 and OCI provider version 4.2.0.

## Input Variables
Input variables used in the configuration are all defined (and defaulted) in config/variables.tf:
- **tenancy_ocid**: the OCI tenancy id where this configuration will be executed. This information can be obtained in OCI Console.
	- Required, no default
- **user_ocid**: the OCI user id that will execute this configuration. This information can be obtained in OCI Console. The user must have the necessary privileges to provision the resources.
	- Required, no default
- **fingerprint**: the user's public key fingerprint. This information can be obtained in OCI Console.
	- Required, no default
- **private_key_path**: the local path to the user private key.
	- Required, no default
- **private_key_password**: the private key password, if any.
	- Optional, default ""
- **home_region**: the tenancy home region identifier where Terraform should provision IAM resources (for a list of available regions, please see https://docs.cloud.oracle.com/en-us/iaas/Content/General/Concepts/regions.htm)
	- Required, no default	
- **region**: the tenancy region identifier where the Terraform should provision the resources.
	- Required, no default
- **region_key**: the 3-letter region key
	- Required, no default
- **service_label**: a label that is used as a prefix when naming provisioned resources.
	- Required, no default
- **vcn_cidr**: the VCN CIDR block
	- Optional, default 10.0.0.0/16
- **public_subnet_cidr**: the public subnet CIDR block.
	- Optional, default 10.0.1.0/24
- **private_subnet_app_cidr**: the App private subnet CIDR block.
	- Optional, default 10.0.2.0/24
- **private_subnet_db_cidr**: the DB private subnet CIDR block.
	- Optional, default 10.0.3.0/24
- **public_src_bastion_cidr**: the external CIDR block that is allowed to ingress into the bastions servers in the public subnet.
	- Required, no default
- **public_src_lbr_cidr**: the external CIDR block that is allowed to ingress into the load balancer in the public subnet.
	- Optional, default 0.0.0.0/0
- **is_vcn_onprem_connected**: whether the VCN is connected to on-premises, in which case a DRG is created and attached to the VCN.
	- Required, default false	
- **onprem_cidr**: the on-premises CIDR block. Only used if is_vcn_onprem_connected == true
	- Optional, default 0.0.0.0/0	
- **network_admin_email_endpoint**: an email to receive notifications for network related events.
	- Required, no default
- **security_admin_email_endpoint**: an email to receive notifications for security related events.
	- Required, no default
- **cloud_guard_configuration_status**: whether Cloud Guard is enabled or not.
	- Optional, default ENABLED
- **cloud_guard_configuration_self_manage_resources**: whether Cloud Guard should seed Oracle-managed entities. Setting this variable to true lets the user seed the Oracle-managed entities with minimal changes to the original entities.
	- Optional, default false

## How to Execute the Code Using Terraform CLI
Within the config folder, provide variable values in the existing *quickstart-input.tfvars* file.

Next, within the config folder, execute:

	terraform init
	terraform plan -var-file="quickstart-input.tfvars" -out plan.out
	terraform apply -var-file="quickstart-input.tfvars" plan.out

Alternatively, rename *quickstart-input.tfvars* file to *terraform.tfvars* and execute:	

	terraform init
	terraform plan -out plan.out
	terraform apply plan.out

## How to Execute the Code Using OCI Resource Manager
There are a few different ways of running Terraform code in OCI Resource Manager (ORM). Here we describe two of them: 
- creating an ORM stack by uploading a folder to ORM;
- creating an ORM stack by integrating with GitLab. 

A stack is the ORM term for a Terraform configuration. Regardless of the chosen method, **an ORM stack must not be contain any state file or *.terraform* folder in Terraform working folder (the *config* folder in this setup)**.

For more ORM information, please see https://docs.cloud.oracle.com/en-us/iaas/Content/ResourceManager/Concepts/resourcemanager.htm.

### Stack from Folder
Create a folder in your local computer (name it say 'cis-oci') and paste there the config and modules folders from this project. 

Using OCI Console, navigate to Resource Manager service page and create a stack based on a folder. In the **Create Stack** page:
1. Select **My Configuration** option as the origin of the Terraform configuration.
2. In the **Stack Configuration** area, select the **Folder** option and upload the folder containing both config and modules folder ('cis-oci' in this example).

![Folder Stack](images/FolderStack_1.png)

3. In **Working Directory**, select the config folder ('cis-oci/config' in this example) .
4. In **Name**, give the stack a name or accept the default.
5. In **Create in Compartment** dropdown, select the compartment to store the Stack.
6. In **Terraform Version** dropdown, **make sure to select 0.13.x**.

![Folder Stack](images/FolderStack_2.png)

Once the stack is created, navigate to the stack page and use the **Terraform Actions** button to plan/apply/destroy your configuration.

![Run Stack](images/RunStack.png)

### Stack from GitLab
**Note:** ORM requires the GitLab instance accessible over the Internet.

Using OCI Console, navigate to Resource Manager service page and create a connection to your GitLab instance.

In the **Configuration Source Providers** page, provide the required connection details to your GitLab, including the **GitLab URL** and your GitLab **Personal Access Token**. 

![GitLab Connection](images/GitLabConnection.png)

Next, create a stack based on a source code control system. Using OCI Console, in the **Create Stack** page:
1. Select **Source Code Control System** option as the origin of the Terraform configuration.
2. In the **Stack Configuration** area, select the configured GitLab repository details:
	- The configured GitLab provider
	- The repository name
	- The repository branch
	- For the **Working Directory**, select the 'config' folder.	 
3. In **Name**, give the stack a name or accept the default.
4. In **Create in Compartment** dropdown, select the compartment to store the stack.
5. In **Terraform Version** dropdown, **make sure to select 0.13.x**.

![GitLab Stack](images/GitLabStack.png)

Once the stack is created, navigate to the stack page and use the **Terraform Actions** button to plan/apply/destroy your configuration.

# Known Issues
## Deployment via Resource Manager or Terraform
- Destroying the stack
	- Vaults have a delayed delete of 7 days
	- Compartments may not delete 
	- Tag namespace fails to delete on the first destroy.  Run destroy again to remove.