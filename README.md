# Deploying Three-Tier Web Application in AWS
I deployed a three-tier web application in AWS based on this [workshop](https://catalog.us-east-1.prod.workshops.aws/workshops/85cd2bb2-7f79-4e96-bdee-8078e469752a/en-US). The application displays the transactions data stored in the database. Using the application, a user can enter new transaction values which are  saved into the database. 
## Demo
This is a quick demo showing the deployed application. A higher resolution video can be found [here](https://github.com/NoraEr/aws-three-tier-webapp/blob/main/Demo/AWS_three_tier_webapp_demo.mp4)

https://github.com/NoraEr/aws-three-tier-webapp/assets/121566705/6f287999-f758-4e7f-bd2d-f6434206c931

Demo overview:
- We access the deployed application using the DNS name of the external load balancer endpoint.
- Using the application, we access transactions values which are retrieved from the database and displayed as a table. We enter new transactions values which are written to the database.
- We verify that these new transactions values are written to the database: we connect to the database instance via EC2 private instance using Session Manager. Then, we retrieve the new transactions values from the database table using SQL.


## Architecture overview 
The architecture comprises a VPC with 2 availabilty zones, load balancers and autoscaling group to maintain high availability. The web server is deployed within EC2 instances in a public subnet with access to the internet via Internet Gateway. The application layer and database sits within a private subnet. The application has access to the internet via NAT Gateway. The database is only accessible via the application layer based on security group inbound rules.

![architecture](https://github.com/NoraEr/aws-three-tier-webapp/blob/main/architecture/three-tier-webapp-architecture.png)

## Deployment steps
- Setup: Created S3 bucket. Uploaded code for the application into S3. Setup IAM role required by EC2 instance.
- Networking: Created a custom VPC with public and private subnets, internet gateway, NAT gateway, route tables and security groups.
- Database Deployment: Deployed Amazon RDS MySQL Aurora database with a read replica for high availability. Configured instance by creating a database table.
- Web server and application deployment: Configured EC2 instance via Session manager. Deployed EC2 instances using Amazon Machine Image (AMI). Attached the appropriate IAM role and security group.
- Load balancing and auto-scaling: To maintain high availability, configured external and internal application load balancers. To ensure horizontal scalability, deployed auto-scaling groups with a minimum capacity of two instances in the web and application layers.

## Testing
- Auto-scaling group: tested that the auto-scaling group maintains the minimum desired capacity of two EC2 instances. This was done by manually deleting one of the two EC2 instances and checking that a new instance is automatically deployed.
- Database: verified that the application layer can connect to the database, read and write data. 
- Tested entire architecture via external load balancer: accessed web server via external load balancer DNS endpoint (as shown in demo). Added data using the front-end interface and checked that this data was saved into the database using the application layer EC2 private instance.

## Lessons learnt
When deploying this end-to-end application, I faced some issues which I have fixed and learnt from. I summarise the lessons learnt below which can be helpful for future projects or for anyone who want to build a similar application:
- There was some issue with accessing the deployed application. Upon investigating this issue, it turns out that I have accidentally attached the wrong security group to the external load balancer. **Always check that the right security group is configured for each component. Make sure to use clear and consistent naming convention to help differentiate between different AWS components.**
- Even after attaching the right security group, the application was still not accessible via the browser. By checking the security group inbound rules, I verified that my public IP address was allowed access. I also checked that my public IP address is correct. However, it turns out that my public IP address has changed the day after I configured the security group. This means the application was not accessible using my new public IP address. After changing the security group inbound rule with my new IP address, I successfully accessed the application as shown in the demo above. For some reason, my Internet Service Provider have changed my public IP address. **Always check your public IP address and make sure it is allowed via the security group inbound rule.**
- The deployment was carried out manually through the AWS Console. It would be helpful to automate the creation, deployment and deletion of resources using Infrastructure as Code with tools such as Terraform.
- Knowing how to test individual components as well as the overall system is key for troubleshooting errors. Often, problems occurs due to networking or security configurations.
- **Always make sure all resources are deployed in the correct VPC and region.** When deploying a resource, the default VPC is attached by default. Always select the custom VPC if there is one and select the right region. If using the console, some critical parameters (such as IAM role for EC2) may be hidden within advanced options. 
- To minimize cost associated with deploying AWS resources, I verifed how much each resource cost. Many resources were not included in the Free Tier. I used a cheaper EC2 instance type (different from the AWS workshop). I also stopped temporarily the database whenever I am not using it. The NAT gateway can be expensive if it is running for a long period. There is no way to stop the NAT gateway. However, the latter can be deleted and re-deployed when needed. If an auto-scaling group is deployed, it's not helpful to stop EC2 instances as the auto-scaling group will automatically spin a new one. To minimise cost, one can temporarily delete the auto-scaling group before stopping EC2 instances. To clean up all resources, this should be done in a specific order due to dependencies between components. When deleting multiple components sequentially that are dependent one another, the console page may need to be refreshed.
  

