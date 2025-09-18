# Brief introduction of terms

* VPC - virtual private cloud. VPC is a network created for your project or organization in aws. VPC will handle the network requirements e.g subnets and ip addresses for your resources.


* CloudFront - Speeds up content delivery by caching data at edge locations worldwide

* Route 53 - performs domain registration, DNS routing and health checks

* Simple Standard Storage (S3) – AWS native storage solution.

* Identity Access Manager (IAM) – IAM controls who has access to what resource(s)

* Elastic Compute Cloud (EC2) – this service provisions a virtual computer instance which will serve as our server.

* Availability Zone - this shows the presence of an aws datacenter within a region. The closer you are to an AZ, the faster your data can be transferred (low latency).

* Cloud Trail – AWS tool to monitor latency between your resources.

Security Group – controls network access to your resource eg EC2 instance

---

### Deployment process - This is a simple overview of the deployment process.

👉 Think of this **flow order**:

**User → Route 53 → CloudFront → Origin (S3) → EC2**


IAM and Security Groups will be used to secure user and network access, and resources can be distributed across availability zones.


 ### Basic S3 static Deployment Architecture
  
  ![basic flow architecture](./images/aws%20basic%20deployment%20archi.png)




### 1. **Route 53 (DNS)**

* Purpose: Acts as the Domain Name System (DNS) for your domain (e.g., `aws-uniuyo-club.com`).
DNS service: it performs domain registration, DNS routing and health checks
(https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/Welcome.html)

* Flow:

  * When a User enters `www.aws-uniuyo-club.com` on his browser→ Route 53 resolves the domain and routes the traffic.
  * Route 53 can route traffic directly to:

    * A CloudFront distribution
    * An Application Load Balancer (ALB)
    * An EC2 public endpoint
    * An S3 bucket (for static hosting).


### 2. **CloudFront Distribution (CDN Layer)**

* Purpose: Speeds up content delivery by caching data at edge locations worldwide.
AWS content delivery network. Manages caching of data at edge locations
around the world to help improve content distribution to users. It basically stores a copy
of your data and forwards this to the user upon request instead of always placing a request to
your backend. 

* Origin Options: We'll be focusing on S3 bucket

  * **S3 bucket** (static site, images, JS, CSS, videos).
  * **ALB or EC2 instance(s)** (dynamic content, APIs).
* Flow:

  * User request → CloudFront edge location.
  * If cached: served immediately.
  * If not cached: CloudFront fetches from origin (S3, EC2, or ALB).


### 3. **EC2 Instances (Compute Layer)**

* Purpose: Run web servers, applications, or APIs.
* Placement:

  * Launched inside a **VPC**.
  * Spread across **subnets** (public or private).
  * Public-facing workloads usually sit behind a Load Balancer in public subnets.
* Flow:

  * Requests (from CloudFront or ALB) → reach EC2 instances.
  * EC2 responds with dynamic data or files stored on S3.


### 4. **S3 Buckets (Storage Layer)**
* Simple standard storage : S3 classes are S3 standard and S3 glacier 
* Purpose: Store static content, backups, logs, or even host static websites.
* Use Cases:

  * CloudFront origin for static assets.
  * Store media/files uploaded via EC2 or applications.
* Flow:

  * CloudFront fetches static files → caches them globally.
  * EC2 can read/write to S3 for storage.


### 5. Connections / Data Flows

* **Route 53 → CloudFront / ALB**: DNS resolves the request.
* **CloudFront → S3 / EC2 / ALB**: CloudFront pulls content from origin.
* **EC2 → S3**: EC2 can fetch/store objects.
* **Clients → CloudFront**: Users connect securely via HTTPS.


### 6. IAM and Network Security 

* IAM controls who has access to what resource(s)

* **IAM Roles**:

  * EC2 instances may have roles granting S3 access.
  * CloudFront may have an Origin Access Identity (OAI) to securely fetch from S3.

* **Security Groups**: Control inbound/outbound traffic for EC2/ALB.
* **Encryption**:

  * HTTPS (TLS) for data in transit.
  * S3 server-side encryption (SSE-S3, SSE-KMS).
  * Encrypted EBS volumes for EC2.


### 7. Availability Zones / Regions

* **Regions**: Geographical locations (e.g., `us-east-1`).
* **Availability Zones (AZs)**: Multiple data centers inside each region.
* Best practice: Deploy EC2 instances or ALBs across **multiple AZs** for high availability.
* CloudFront: Automatically leverages multiple edge locations globally.
* S3: Region-wide service (data replicated across multiple AZs within the region).


---


# Frontend Deployment to S3
Why S3?

S3 is designed for static website hosting (HTML, CSS, JS).
Highly available and scalable, no need to manage servers.
Cost-effective for serving static assets.

### Steps:
1. Create an S3 bucket → name it the same as your domain (e.g., example.com).

2. Enable static website hosting in the bucket properties.

3. Upload your frontend build files (from GitHub → build locally → upload).

4. Set bucket policy/public access to allow GetObject (or later secure via CloudFront).

* Note the S3 website endpoint URL (e.g., http://aws-club-bucket02.s3-website-us-east-1.amazonaws.com).

# Setup custom DNS and routing 
### CDN with Cloudfront
Why CloudFront?

* It Caches content at edge locations, improving global performance.

* Adds HTTPS support (with AWS Certificate Manager).

* Protects S3 from direct public exposure.

### Steps:

1. On AWS console, go to CloudFront → Create Distribution.

2. Choose your S3 bucket as the origin.

3. Enable Redirect HTTP → HTTPS.

4. Attach an SSL/TLS certificate from AWS Certificate Manager (provisioned for your domain).

* Note your CloudFront distribution domain.

# Route with route53
Why Route 53?

Route 53 manages custom domains (DNS) and routing to AWS resources.
### Steps:

1. In Route 53 → Hosted Zones, create a hosted zone for your domain.

2. Add an A record (alias) → point it to the CloudFront distribution.

3. Verify domain registration/DNS propagation.

# Deploy Backend API to EC2
Why not deploy backend to S3?

S3 only serves static files, not dynamic APIs (S3 cannot handle server-side processing). Backend APIs need compute (EC2, Lambda, or EKS).

### Steps:

1. On your console, naviagte to EC2 → Launch an EC2 instance (Select Ubuntu).

2. Select your VPC, subnet, and security group (allow HTTP/HTTPS, SSH). Launch the instance (Give it a minute or two to get ready)

3. Connect to your EC2 instance via EC2 Instance Connect (or SSH).

4. Once inside your EC2 instance, install your runtime (e.g Node.js.) and clone your GitHub repo:

```bash
git clone https://github.com/chilet-okeke/ec2-backend.git
cd app && npm install && npm start

```
* Note: Step 4 can be improved by implementing a startup script which will automatically execute these commands, reducing the need for you to connect to the instance manually. This can prove to be very helpful at scale.

5. Test the backend API with the instance public ip address with curl or on your browser.

```bash
curl <ec2-external-ip e.g 172.3.45.33>
```

# Connect backend to frontend
1. Update frontend API calls (e.g., fetch()) to use the EC2 backend endpoint.

2. Ensure backend also has HTTPS (via Load Balancer + ACM or self-managed cert).

3. Secure connection with:

* IAM roles (S3 access for backend if needed).

* Security Groups (only allow CloudFront to access EC2, not the whole internet).

# Monitor with Cloudtrail and CloudWatch
Why CloudTrail?

CloudTrail records all API calls and actions in your AWS account It helps detect unusual behavior and troubleshoot latency/security issues.

### Steps:

1. Open CloudTrail → Trails → Create Trail.

2. Apply to All regions.

3. Choose an S3 bucket for log storage.

4. Enable CloudWatch integration for monitoring events in real-time.

5. Use CloudWatch Metrics & Alarms to track API performance and latency.

---

## FAQS and Open Questions

* Can I setup a full-stack app in EC2 without using S3?

Yes, you can deploy both frontend and back end on EC2 as well as the other compute options such as AWS Lambda and EKS

* Is AWS expensive to use?

AWS costs depends on usage, costs up with increase in traffic, more powerful compute provisioning and storage usage. It is recommended you use only the necessary resources, smaller compute instance sizes and utilize optimization best practices to keep costs down.

* Can I migrate a pre-existing fullstack app to aws?
Yes, migration to AWS cloud is common, you can use a similar setup with this deployment session to pull from your application repo, or use Application Migration Service (MGN) to migrate your application.


# Some Helpful links

EC2 basics
(https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/concepts.html)

Create static frontend route with s3 bucket
(https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/getting-started-s3.html)

Cloudfront distribution from s3 origin
(https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/GettingStarted.SimpleDistribution.html)



