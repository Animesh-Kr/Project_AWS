### AWS Cloud Infrastructure Project: Scalable Web Hosting with VPC, EC2, and Elastic Load Balancing

#### Project Overview
This project involved designing and implementing a secure, scalable, and highly available cloud infrastructure on Amazon Web Services (AWS) to host multiple web applications. By leveraging AWS Virtual Private Cloud (VPC), Elastic Compute Cloud (EC2), and Elastic Load Balancing (ELB), I created a robust environment capable of handling dynamic traffic patterns, ensuring fault tolerance, and optimizing resource utilization. The infrastructure was configured using the AWS Management Console, with automation scripts for reproducibility, and integrated best practices for security and performance.

#### Key Components and Implementation Details

1. **Virtual Private Cloud (VPC) Setup**
   - **Objective**: Create an isolated network environment to securely host resources and control traffic flow.
   - **Implementation**:
     - Configured a custom VPC with a CIDR block (e.g., `10.0.0.0/16`) to define the IP address range.
     - Divided the VPC into public and private subnets across at least two Availability Zones (AZs) for high availability:
       - Public subnets (e.g., `10.0.1.0/24`, `10.0.2.0/24`) for resources like load balancers and NAT gateways.
       - Private subnets (e.g., `10.0.3.0/24`, `10.0.4.0/24`) for EC2 instances hosting application servers.
     - Set up an **Internet Gateway** attached to the VPC to enable communication between public subnets and the internet.
     - Configured **Route Tables**:
       - Public route table with a route to the Internet Gateway (`0.0.0.0/0 -> igw-id`).
       - Private route table with a route to a NAT Gateway (`0.0.0.0/0 -> nat-id`) for outbound internet access from private subnets.
     - Implemented **Network Access Control Lists (NACLs)** and **Security Groups** to enforce network-level and instance-level security, allowing only necessary traffic (e.g., HTTP on port 80, SSH on port 22 from specific IP ranges).
     - Enabled **VPC Flow Logs** to monitor and troubleshoot network traffic, stored in Amazon CloudWatch for analysis.
   - **Outcome**: A secure, segmented network infrastructure ensuring isolation of resources, controlled access, and fault tolerance across multiple AZs.

2. **Elastic Compute Cloud (EC2) Configuration**
   - **Objective**: Deploy virtual servers to host web applications with scalability and reliability.
   - **Implementation**:
     - Launched multiple EC2 instances (e.g., `t2.micro` for cost efficiency during testing) in private subnets across two AZs to ensure high availability.
     - Selected an Amazon Machine Image (AMI) with a web server stack (e.g., Amazon Linux 2 with Apache or Nginx pre-installed).
     - Configured **User Data** scripts to automate the installation and configuration of web servers and application dependencies upon instance launch:
       ```bash
       #!/bin/bash
       yum update -y
       yum install httpd -y
       systemctl start httpd
       systemctl enable httpd
       echo "<h1>Welcome to My Web App</h1>" > /var/www/html/index.html
       ```
     - Assigned **IAM Roles** to EC2 instances to securely access AWS services (e.g., Amazon S3 for static assets) without hardcoding credentials.
     - Configured **Security Groups** to allow:
       - Inbound HTTP (port 80) from the load balancer’s security group.
       - Inbound SSH (port 22) from a bastion host or specific admin IPs for secure management.
     - Integrated EC2 instances with **Amazon EC2 Auto Scaling** to automatically adjust the number of instances based on CPU utilization or traffic demand, ensuring scalability.
     - Used **Elastic Block Store (EBS)** volumes for persistent storage, with snapshots backed up to Amazon S3 for data durability.
   - **Outcome**: A fleet of EC2 instances running web applications, automatically scalable, secure, and resilient to failures.

3. **Elastic Load Balancing (Application Load Balancer)**
   - **Objective**: Distribute incoming traffic across EC2 instances to ensure high availability, fault tolerance, and optimal performance.
   - **Implementation**:
     - Created an **Application Load Balancer (ALB)** in the public subnets, configured as internet-facing to handle HTTP/HTTPS traffic.
     - Defined **Listeners**:
       - HTTP listener on port 80 to accept client requests.
       - (Optional) HTTPS listener on port 443 with an SSL certificate managed via AWS Certificate Manager (ACM) for secure communication.
     - Set up **Target Groups** to route traffic to registered EC2 instances:
       - Target type: Instance, protocol: HTTP, port: 80.
       - Configured health checks (e.g., path `/health`, HTTP 200 status) to monitor instance health, ensuring traffic is routed only to healthy instances.
     - Registered EC2 instances from both AZs with the target group to balance traffic evenly.
     - Enabled **Cross-Zone Load Balancing** to distribute traffic across all registered instances, regardless of their AZ, improving fault tolerance.
     - Configured **Listener Rules** to support advanced routing (e.g., path-based routing for `/app1` to one target group and `/app2` to another).
     - Integrated with **Amazon CloudWatch** to monitor ALB metrics (e.g., request count, latency, error rates) and set up alarms for proactive issue detection.
     - Tested load balancing by accessing the ALB’s DNS name in a browser, verifying traffic distribution across instances.
   - **Outcome**: A highly available load balancer efficiently distributing traffic, improving application performance, and ensuring no single point of failure.

#### Automation and Reproducibility
- **Infrastructure as Code (IaC)**:
  - Used **AWS CloudFormation** templates to define and provision VPC, subnets, route tables, EC2 instances, and ALB resources, ensuring consistent deployments.
  - Example CloudFormation snippet for ALB:
    ```yaml
    Resources:
      MyALB:
        Type: AWS::ElasticLoadBalancingV2::LoadBalancer
        Properties:
          Subnets:
            - !Ref PublicSubnet1
            - !Ref PublicSubnet2
          SecurityGroups:
            - !Ref ALBSecurityGroup
          Scheme: internet-facing
          Type: application
    ```
  - Stored templates in a GitHub repository for version control and collaboration.
- **Scripting**:
  - Wrote Bash scripts for EC2 instance configuration and deployment tasks, stored in the repository.
  - Used AWS CLI commands to automate resource management (e.g., registering instances with target groups).

#### Security Best Practices
- Restricted access using security groups and NACLs to minimize attack surfaces.
- Enabled encryption for data in transit (HTTPS via ALB) and at rest (EBS volumes).
- Used IAM roles and policies to follow the principle of least privilege.
- Configured CloudWatch Logs and VPC Flow Logs for auditing and monitoring.

#### Challenges and Solutions
- **Challenge**: Ensuring seamless traffic distribution during instance scaling.
  - **Solution**: Integrated Auto Scaling with ALB target groups, enabling automatic registration/deregistration of instances.
- **Challenge**: Securing SSH access to private EC2 instances.
  - **Solution**: Deployed a bastion host in a public subnet, allowing SSH access only from specific IPs through the bastion.
- **Challenge**: Managing costs during development.
  - **Solution**: Utilized AWS Free Tier resources (e.g., t2.micro instances) and set up CloudWatch billing alarms to monitor usage.

#### Outcomes and Impact
- Deployed a scalable infrastructure capable of hosting multiple web applications with minimal downtime.
- Achieved high availability by distributing resources across multiple AZs, with the ALB ensuring traffic is routed only to healthy instances.
- Improved performance through efficient load balancing and auto-scaling, handling traffic spikes seamlessly.
- Enhanced security with isolated networks, encrypted traffic, and fine-grained access controls.
- Demonstrated proficiency in AWS services, IaC, and cloud architecture best practices, suitable for enterprise-grade applications.

#### Technologies Used
- **AWS Services**: VPC, EC2, ELB (Application Load Balancer), Auto Scaling, CloudWatch, IAM, ACM, EBS, S3
- **Tools**: AWS Management Console, AWS CLI, CloudFormation, Git, Bash
- **Protocols/Standards**: HTTP/HTTPS, TCP, CIDR, OSI Model (Layer 7 for ALB)

#### GitHub Repository
- The project’s CloudFormation templates, scripts, and documentation are available in my [GitHub repository](https://github.com/yourusername/aws-web-infrastructure) (replace with your actual repo link).
- Includes detailed README with setup instructions, architecture diagrams, and deployment steps for reproducibility.

#### Future Enhancements
- Integrate **AWS Route 53** for custom domain management and DNS routing.
- Implement **AWS WAF** to protect against common web exploits.
- Containerize applications using **Amazon ECS** or **EKS** for improved portability and management.
- Explore **AWS Lambda** for serverless components to reduce operational overhead.

---

### Why Include This in Your GitHub Profile?
- **Showcases Technical Expertise**: Demonstrates hands-on experience with core AWS services and cloud architecture principles.
- **Highlights Problem-Solving**: Details challenges faced and solutions implemented, showcasing critical thinking.
- **Emphasizes Best Practices**: Reflects adherence to security, scalability, and automation standards.
- **Portfolio Appeal**: Provides a tangible project with a GitHub repository link, allowing potential employers or collaborators to review your code and documentation.

To add this to your GitHub profile:
1. Create a repository for the project and push your CloudFormation templates, scripts, and a detailed README.
2. Pin the repository to your GitHub profile for visibility.
3. Add a summary of this project in your profile’s README under a “Featured Projects” section, linking to the repository.

This detailed explanation serves as both a technical narrative for your GitHub profile and a portfolio piece to impress recruiters or collaborators. Let me know if you need help with drafting the GitHub README, creating architecture diagrams, or generating specific CloudFormation templates![](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/create-application-load-balancer.html)[](https://medium.com/%40sayalishewale12/setting-up-an-application-load-balancer-with-aws-ec2-39f7a74d89a)[](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/application-load-balancer-getting-started.html)
