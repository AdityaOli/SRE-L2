# Infrastructure

## AWS Zones
Primary : ``us-east-2``

Secondary/DR : ``us-west-1``

## Servers and Clusters

### Table 1.1 Summary

| Asset      | Purpose           | Size                                                                   | Qty                                                             | DR                                                                                                           |
|------------|-------------------|------------------------------------------------------------------------|-----------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------|
| Git | A location to store the code and the configuration to provision the infra | - | 1 | Deployed to the active ``us-east-2`` |
| Git | A location to store the code and the configuration to provision the infra | - | 1 | Deployed to the DR site ``us-west-1`` (passive/turned off) |
| Load Balancer | Different amazon EC2 instances would be hosted across AZs. A load balancer is needed to distribute traffic across those EC2s. | - | - | Deployed to the active ``us-east-2`` |
| Load Balancer | Different amazon EC2 instances would be hosted across AZs. A load balancer is needed to distribute traffic across those EC2s. | - | - | Deployed to the DR site ``us-west-1`` |
| VPC | VPC would store multiple floating virtual IPs for us to use. Because of which traffic from LB could be sent to the EC2s. | - | - | Deployed to the active ``us-east-2`` |
| VPC | VPC would store multiple floating virtual IPs for us to use. Because of which traffic from LB could be sent to the EC2s. | - | - | Deployed to the DR site ``us-west-1`` |
| EC2 | Application would be running on a linux server which is EC2. These need to be present in multiple AZs | t3.micro | 3 (2 minimum) | Deployed to the active ``us-east-2`` |
| EC2 | Application would be running on a linux server which is EC2. These need to be present in multiple AZs | t3.micro | 3 (2 minimum) | Deployed to the DR site ``us-west-1`` |
| Kubernetes (EKS) | K8s cluster for where prometheus and grafana would reside | t3.medium | 2 nodes per cluster | Deployed to the active ``us-east-2`` |
| Kubernetes (EKS) | K8s cluster for where prometheus and grafana would reside | t3.medium | 2 nodes per cluster | Deployed to the DR site ``us-west-1`` |
| S3 | Used to host the terraform state file | - | 1 | Deployed to the primary site ``us-east-2`` |
| S3 | Used to host the terraform state file | - | 1 | Deployed to the DR site ``us-west-1`` |
| RDS | RDS service where the DB app would run | db.t2.small | 2 nodes per cluster | Deployed to the primary site ``us-east-2`` |
| RDS DB | RDS Database where the data would be stored | db.t2.small | 2 nodes per cluster | Deployed to the DR site ``us-west-1`` |


### Descriptions

Git : Any code would need a version management system to efficiently store and manage the code. This would be run as active and turned off in the DR site only to be used as a DR not as HA.
Load Balancer : A load balander is used to distribute load among the applications it is connected to (EC2 here). So when a region is down, an EC2 in another region would still be able to support the system because the laod balancer would simply mark the lost EC2 as inactive and not send traffic to it, thereby enhancing HA. SO one would be used in active system while the ther would be abel to help in DR scenario on the DR system.
VPC : A VPC would help in storing IPs and create a private cloud env for the EC2 instances running inside it. The EC2 would not need to specifically remember the IPs and neither would the DNS. Multiple IPs from different AZs are used to support HA and DR.
EC2 : These would help in storing the code for the application which would then send its data to be stored. Invoking this application would enable the use of monitoring the APIs on the prometheus monitoring stack. Hence 3 EC2 to maintain HA quorum, and a replica of this system to maintain DR.
EKS : EKS would be the elastic kubernetes service hosting kubernetes. K8s would then host prometheus and grafana which are being used for monitoring.
S3 : These would store your extremely important state file which store the state of the end system.
RDS : This wouls run the DB application and hence is needed for communication with the database. In each region, we would need two clusters to satisfy the HA and DR needs.
RDS DB : The database where all the data would be stored. 2 nodes per cluster each on the primary and DR site.

## DR Plan
### Pre-Steps:
* Access to git
* Primary and DR have no configuration drift
* Deploy the infra to the DR site and disable the primary site to avoid inconsistency until primary is fixed.

## Steps:
Infra Deploy through CI/CD
Change DNS entries to point to DR Load Balancer. 
Create the DR site as primary and brgin data duplication on DB from DR to primary. 
Point monitoring to monitor DR site.