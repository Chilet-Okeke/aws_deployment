# Brief introduction of terms

VPC - virtual private cloud. VPC is a network created for your project or organization in aws. VPC will handle the network requirements e.g subnets and ip addresses for your resources.


CloudFront - Speeds up content delivery by caching data at edge locations worldwide

Route 53 - performs domain registration, DNS routing and health checks

Simple Standard Storage (S3) – AWS native storage solution.

Identity Access Manager (IAM) – IAM controls who has access to what resource(s)

Elastic Compute Cloud (EC2) – this service provisions a virtual computer instance which will serve as our server.

Availability Zone - this shows the presence of an aws datacenter within a region. The closer you are to an AZ, the faster your data can be transferred (low latency).

Cloud Trail – AWS tool to monitor latency between your resources.

Security Group – controls network access to your resource eg EC2 instance

---

### Deployment process - This is a simple overview of the deployment process.

👉 Think of this **flow order**:

**User → Route 53 → CloudFront → Origin (S3) → EC2**


IAM and Security Groups will be used to secure user and network access, and resources can be distributed across availability zones.


 ### Basic Deployment Architecture
  
  ![basic flow architecture](./aws%20basic%20deployment%20archi.png)




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
Explain why you deployed static frontend to S3

# Setup custom DNS and routing 
CDN with Cloudfront
Route with route53

# Backend Deployment to EC2
Explain why You can't deploy backend to S3
Deploy backend from github repo to EC2


# Connect backend to frontend

# Monitor latency with Cloudtrail
If there's still time monitor system health with cloudwatch

---

# Brief Q&A session

## FAQS

* Can I setup a full-stack app in EC2 without using S3?
* Is AWS expensive to use?
* Can I migrate a pre-existing fullstack app to aws?

## Open questions



# Helpful links

EC2 basics
(https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/concepts.html)

Create static frontend route with s3 bucket
(https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/getting-started-s3.html)

Cloudfront distribution from s3 origin
(https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/GettingStarted.SimpleDistribution.html)



