# AWS to Azure Server Migration Lab

## Video Walkthrough

This project walkthrough is divided into two videos:

- 🎥 [Part 1 — AWS Source Environment and Azure Migration Setup](https://www.loom.com/share/bbfc32cb320c42d0b4b3d28e93adb615)
- 🎥 [Part 2 — Azure Migrate Configuration, Validation, and Project Review](https://www.loom.com/share/8fd48238e26448a7917c638fe98ddfce)

## Project Overview

This hands-on lab demonstrates how to discover, assess, replicate, test, and migrate a Windows Server 2022 virtual machine from Amazon Web Services (AWS) to Microsoft Azure using Azure Migrate.

The project was designed to simulate a real infrastructure migration while documenting the networking, identity, discovery, replication, validation, and troubleshooting work required during a cloud-to-cloud move.

## Project Objectives

- Build a Windows Server 2022 source environment in AWS.
- Configure the required AWS networking and IAM permissions.
- Build the destination network and resource groups in Azure.
- Deploy and register an Azure Migrate appliance.
- Discover and assess the AWS EC2 server.
- Configure replication to Azure.
- Validate the workload through a test migration.
- Complete the migration and verify the Azure virtual machine.

## Architecture

### AWS Source Environment

| Resource | Configuration |
| --- | --- |
| Region | `us-east-1` |
| VPC | `10.0.0.0/16` |
| Subnet | `10.0.1.0/24` |
| EC2 instance | `vm-migrate-source-nabil` |
| Operating system | Windows Server 2022 |
| Security group | `migrate-source-nabil` |
| WinRM rule | TCP `5985` from the Azure appliance public IP only |
| IAM role | `role-azure-migrate-nabil` |
| IAM policy | `policy-azure-migrate-nabil` |
| Instance profile | `profile-azure-migrate-nabil` |

### Azure Destination Environment

| Resource | Configuration |
| --- | --- |
| Region | East US |
| Source resource group | `rg-migrate-source-nabil` |
| Target resource group | `rg-migrate-target-nabil` |
| Virtual network | `vnet-migrate-nabil` — `10.1.0.0/16` |
| Subnet | `snet-migrate` — `10.1.1.0/24` |
| Azure Migrate project | `migrate-project-nabil` |
| Discovery appliance | `appliance-migrate-nabil` |
| Appliance VM | `vm-mig-appl-nabil` |
| Appliance computer name | `migappl-nabil` |
| Appliance size | `Standard_D8s_v4` — 8 vCPU, 32 GB RAM |
| Additional services | Replication cache storage and Recovery Services Vault |

## Migration Workflow

### 1. Build the AWS Source Environment

I created an AWS VPC and subnet, attached an internet gateway, configured the route table, and deployed a Windows Server 2022 EC2 instance. I then created the IAM role, policy, and instance profile required for Azure Migrate discovery.

### 2. Prepare the Azure Destination

I created separate Azure resource groups for migration infrastructure and the migrated workload. I also deployed the target virtual network and subnet that would host the migration components and destination virtual machine.

### 3. Deploy the Azure Migrate Appliance

I created `migrate-project-nabil`, deployed the discovery appliance, and registered it with the Azure Migrate project. The final appliance used the `Standard_D8s_v4` size after the originally selected VM sizes were unavailable in East US.

### 4. Configure Credentials and Connectivity

I added the AWS access credentials and the local Windows `Administrator` credentials to the appliance. For Windows discovery, I allowed TCP port `5985` from the appliance public IP to the AWS source security group and used WinRM over HTTP for the lab.

> Security note: Credentials, passwords, subscription IDs, access keys, and public IP addresses are intentionally excluded from this repository.

### 5. Discover and Assess the EC2 Server

I added the AWS EC2 source using its IP address and verified that the appliance could communicate with the server. After discovery completed, I created an Azure Migrate assessment to review readiness, sizing, and compatibility. The server assessment returned **Ready for Azure**.

### 6. Configure Replication

I selected the destination subscription, `rg-migrate-target-nabil`, target virtual network, subnet, storage, and VM settings. After replication initialized, I monitored the server until it reached the protected state.

### 7. Test and Complete the Migration

I ran a test migration before cutover to verify that the server could start successfully in Azure without affecting the AWS source. After validation, I cleaned up the test migration and initiated the final migration.

### 8. Validate the Azure VM

After cutover, I confirmed that the migrated VM appeared in the target resource group, connected to it through RDP, and verified that the hostname and Windows Server version matched the AWS source.

## Troubleshooting and Lessons Learned

### Azure VM Quota and SKU Availability

The first appliance deployments failed because the subscription had no available `StandardDSv5Family` quota in East US and `Standard_A8_v2` was unavailable. I reviewed regional VM usage, evaluated alternate sizes, and successfully deployed the appliance with `Standard_D8s_v4`.

### EC2 Server Was Not Discovered

When the EC2 server did not appear in Azure Migrate, I re-entered the AWS access key and secret, confirmed that the appliance and source were configured for the same AWS region, and revalidated the discovery source.

### Windows Validation Failed

The Windows source initially failed validation. I limited inbound TCP `5985` to the appliance public IP, disabled HTTPS-only discovery for the lab, allowed WinRM HTTP fallback, and ran validation again successfully.

### Replication Capacity Constraints

The replication workflow required more compute capacity than the default AWS quota allowed. I submitted a vCPU quota request and evaluated a separate replication appliance design while monitoring the request. This reinforced the importance of checking service quotas before beginning a migration window.

## Skills Demonstrated

- Azure Migrate discovery, assessment, replication, and cutover
- AWS EC2, VPC, security groups, IAM roles, and instance profiles
- Azure resource groups, VNets, subnets, storage, and virtual machines
- Windows Server administration, RDP, and WinRM
- Cross-cloud networking and access control
- Capacity planning, quota analysis, and troubleshooting
- Migration testing, validation, and technical documentation

## Security Practices

- Restricted WinRM access to a single appliance IP address.
- Avoided committing credentials or sensitive identifiers to GitHub.
- Used dedicated resource groups to separate migration infrastructure from the target workload.
- Performed a test migration before the production cutover.
- Removed temporary test-migration resources after validation.

## Repository Structure

```text
.
├── README.md
├── terraform/
│   ├── aws/
│   └── azure/
└── screenshots/
    ├── architecture/
    ├── discovery/
    ├── assessment/
    ├── replication/
    └── validation/
```

## Key Takeaway

This project gave me practical experience planning and executing a cross-cloud server migration. The most valuable part of the lab was not only moving the server, but also diagnosing quota, regional capacity, credential, and WinRM connectivity issues across AWS and Azure.
