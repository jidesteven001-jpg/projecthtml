CloudBridge Technologies — AWS Highly Available Website
A responsive static website for CloudBridge Technologies, deployed on AWS using a multi-AZ architecture. This repository contains the website source code and documents the cloud infrastructure used to host and deliver it securely.
Project Overview
The website is built with:
- HTML5 for page structure
- CSS3 for styling and responsive layout
- Vanilla JavaScript for navigation, animations, and interactive elements
The website files include index.html, style.css, script.js, images, and a favicon.
AWS Architecture
The website is hosted on two CentOS EC2 instances distributed across two Availability Zones (AZs), behind a load balancer. Amazon CloudFront provides the public HTTPS entry point, while Route 53 manages the custom domain.
Architecture components
- Amazon VPC (custom VPC): Isolated network for the deployment.
- Four subnets across two Availability Zones: Two public subnets host the web instances (one instance in Public Subnet A in AZ 1 and one in Public Subnet B in AZ 2). The remaining two subnets are available for the intended network design, such as private subnets, if configured.
- Amazon EC2 (CentOS): Two web-server instances running the website.
- Elastic Load Balancing (ELB): Distributes incoming origin traffic across the registered instances. This project assumes an Application Load Balancer (ALB).
- EC2 Target Group: Registers both instances and uses health checks to determine which targets can receive traffic.
- Amazon CloudFront: Serves the site through a global edge network and connects to the load balancer as its origin.
- Amazon Route 53: Hosts the DNS record for the custom domain purchased/managed through Go54.
- AWS Certificate Manager (ACM): Provides the TLS certificate for HTTPS. A CloudFront viewer certificate must be issued/imported in US East (N. Virginia), us-east-1.
- Go54: Domain registrar/provider for the custom domain.
Request flow
Visitor
  |
  | HTTPS — custom domain
  v
Amazon Route 53
  |
  v
Amazon CloudFront  <--- ACM certificate (us-east-1)
  |
  | Origin request
  v
Application Load Balancer (ELB)
  |
  v
Target Group
  |                         |
  v                         v
CentOS EC2 Instance 1    CentOS EC2 Instance 2
Public Subnet A          Public Subnet B
AZ 1                     AZ 2
  |                         |
  +------ Website files ----+
Important: Route 53 does not host website files; it provides DNS routing. CloudFront delivers cached content and forwards requests to its configured origin. For this design, configure CloudFront's origin as the ALB DNS name, and point the Route 53 record for the custom domain to the CloudFront distribution.

Repository Structure
.
├── index.html
├── style.css
├── script.js
├── profile.jpeg
├── flyer1.jpeg
├── flyer2.jpeg
├── assets/
│   └── favicon.png
└── README.md
Deployment Summary
1. Prepare the VPC
1. Create a custom VPC with an appropriate IPv4 CIDR block.
2. Create four subnets across two Availability Zones.
3. Ensure the two web-server subnets have the routing and outbound access required for your chosen design.
4. Attach an Internet Gateway and configure route tables for public subnets.
5. If the other two subnets are intended to be private, keep application instances there only if the load balancer and management access are designed accordingly.
2. Launch the CentOS EC2 instances
1. Launch two CentOS EC2 instances, one in Public Subnet A and one in Public Subnet B.
2. Install and configure a web server such as Apache HTTP Server (httpd) or Nginx on both instances.
3. Copy the website files to the web server's document root.
4. Configure the web server to serve index.html and the related assets.
5. Confirm that both instances serve the site successfully.
3. Configure the load balancer and target group
1. Create a target group for the web instances using the appropriate protocol and port (commonly HTTP port 80 between the ALB and web servers).
2. Register both EC2 instances.
3. Configure a health-check path such as /.
4. Create an internet-facing Application Load Balancer and attach it to public subnets in both Availability Zones.
5. Configure the ALB listener and security groups so the ALB can reach the instances and only intended traffic is allowed.
4. Configure HTTPS with ACM and CloudFront
1. Request or import an ACM certificate for the custom domain.
2. For a certificate attached to CloudFront, the certificate must be in us-east-1.
3. Validate domain ownership using DNS validation.
4. Create a CloudFront distribution with the ALB DNS name as its origin.
5. Add the custom domain as an alternate domain name (CNAME) on CloudFront and attach the ACM certificate.
6. Configure viewer protocol policy to redirect HTTP to HTTPS.
7. Set suitable cache behaviors. For dynamic or frequently updated assets, adjust caching and invalidation practices as needed.
5. Configure Route 53 and the Go54 domain
1. Create or use a public hosted zone in Route 53 for the domain.
2. If Route 53 is authoritative for DNS, update the domain's nameservers at Go54 to the nameservers assigned by Route 53.
3. Create an A/AAAA Alias record (where supported) pointing the root domain to the CloudFront distribution. Create a suitable record for www if required.
4. Wait for DNS propagation and test the domain over HTTPS.
Security and Reliability Considerations
- Allow public web traffic to CloudFront; restrict the origin so the ALB is not unintentionally exposed as an alternate public entry point.
- Allow inbound web traffic to the EC2 instances from the ALB security group rather than from all IP addresses, where possible.
- Use SSH only from a trusted IP address, or prefer AWS Systems Manager Session Manager where configured.
- Do not store AWS access keys, private keys, passwords, or other secrets in this repository.
- Keep CentOS packages and the web server updated. Check the lifecycle/support status of the specific CentOS release you use.
- Configure target-group health checks and review EC2, ALB, and CloudFront logs/metrics.
- Consider using private subnets for the web instances with the ALB in public subnets for a more secure production architecture. That would differ from the described current setup, where both instances are in public subnets.
- CloudFront-to-ALB HTTPS requires an appropriate origin configuration and certificate on the ALB if HTTPS is used for the origin connection.
Validation Checklist
- [ ] Both EC2 instances serve the website.
- [ ] Both instances are healthy in the target group.
- [ ] The ALB forwards requests to healthy targets.
- [ ] CloudFront origin points to the correct ALB DNS name.
- [ ] ACM certificate is validated and attached to CloudFront in us-east-1.
- [ ] Route 53 records point to the CloudFront distribution.
- [ ] Go54 nameservers point to Route 53, if Route 53 is authoritative.
- [ ] The custom domain loads over HTTPS.
- [ ] The site remains available when one target is unhealthy (subject to the remaining target being healthy).
Run Locally
You can preview the static site locally by opening index.html in a browser. For a more realistic local preview, run a simple local HTTP server from the project directory:
python3 -m http.server 8000
Then visit http://localhost:8000.
Project Purpose
This project demonstrates practical cloud deployment concepts including custom VPC networking, multi-AZ EC2 hosting, load balancing, target-group health checks, DNS configuration, TLS certificates, and CloudFront content delivery.
Disclaimer
This README describes the architecture provided for this project. Confirm your actual AWS console configuration before presenting each component as deployed. AWS resource names, region, subnet CIDRs, instance sizes, domain name, and certificate details should be added only when verified.
CloudBridge Technologies — Building the Future on AWS Cloud
