# AWS Three-Tier Web Application
I deployed a three-tier web application in AWS based on this [workshop](https://catalog.us-east-1.prod.workshops.aws/workshops/85cd2bb2-7f79-4e96-bdee-8078e469752a/en-US/introduction/getting-started). The application displays the transactions data stored in the database. Using the application, a user can enter new transaction values which are  saved into the database. 
## Demo
This is a quick demo showing the deployed application. A higher resolution video can be found [here](https://github.com/NoraEr/aws-three-tier-webapp/blob/main/Demo/AWS_three_tier_webapp_demo.mp4)

https://github.com/NoraEr/aws-three-tier-webapp/assets/121566705/6f287999-f758-4e7f-bd2d-f6434206c931

Demo overview:
- We access the deployed application using the DNS name of the external load balancer endpoint.
- Using the application, we access transactions values which are read from the database and displayed as a table. We enter new transactions values which are written to the database.
- We verify that these new transactions values are written to the database: we connect to the database instance via EC2 private instance using Session Manager. Then, we read the new transactions values from the database table.


## Architecture overview 
The architecture comprises a VPC with 2 availabilty zones, load balancers and autoscaling group to maintain high availability. The web server is deployed within EC2 instances in a public subnet with access to the internet via Internet Gateway. The application layer and database sits within a private subnet. The application has access to the internet via NAT Gateway. The database is only accessible via the application layer by configuring its security group.

![architecture](https://github.com/NoraEr/aws-three-tier-webapp/blob/main/architecture/three-tier-webapp-architecture.png)

## Deployment steps
- Setup: Created S3 bucket. Uploaded code for the application into S3. Setup IAM role required by EC2 instance
- Networking: Created a custom VPC with public and private subnets, internet gateway, NAT gateway, route tables and security groups
- Database Deployment: Deployed Amazon RDS MySQL Aurora database with a read replica for high availability. Configured instance by creating a database table.
- Web server and application deployment: Configured EC2 instance via Session manager. Deployed EC2 instances using Amazon Machine Image (AMI). Attached the appropriate IAM role and security group.
- Load balancing and auto-scaling: To maintain high availability, configured external and internal application load balancers. To ensure horizontal scalability, deployed auto-scaling groups with a minimum capacity of two instances.

