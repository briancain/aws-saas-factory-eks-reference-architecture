# ResilienceHub V2 Assessment Findings

**Assessment Date:** 2026-01-22  
**App Name:** eks-saas-reference-architecture-20260122  
**App ARN:** arn:aws:resiliencehub:us-west-2:575754814836:app/99890b5f-9561-4f35-91b5-45d1215b6078  
**Assessment ID:** dc979e08-3f7f-4e9c-a380-02db72210ea5  
**Resilience Score:** 58/100  
**Total Findings:** 7

---

## High Severity Findings (3)

### 1. EKS microservices running single pod replicas create service SPOFs
- **Finding ID:** 92a5ebbc-0b86-47c9-945b-18e914d752ee
- **Severity:** High
- **Status:** Open
- **Failure Category:** SinglePointOfFailure
- **User Journey:** journey-2
- **Description:** Both the product and order services are running with only one pod replica each (product-57f69dfddc-b48x4 and order-6bd57b4694-tk7ks). If either pod crashes, gets evicted, or the hosting node fails, the corresponding service becomes completely unavailable. This directly impacts the Place Orders journey as customers cannot complete purchases, and the Browse Products journey as customers cannot view the product catalog.

### 2. EKS microservices lack pod topology spread constraints for AZ distribution
- **Finding ID:** df4f4814-3035-4f24-8d5b-dbfac6b457b2
- **Severity:** High
- **Status:** Open
- **Failure Category:** SinglePointOfFailure
- **User Journey:** journey-1
- **Description:** The product and order service deployments in EKS do not have pod topology spread constraints configured. Currently, each service has only one pod (product-57f69dfddc-b48x4 and order-6bd57b4694-tk7ks) which could be scheduled in the same availability zone. If that AZ fails, both the Browse Products and Place Orders journeys become unavailable, preventing customers from viewing products or completing purchases.

### 3. Single-region deployment creates regional SPOF for entire SaaS platform
- **Finding ID:** e1599759-1e08-4e8e-893a-41ce7f513325
- **Severity:** High
- **Status:** Open
- **Failure Category:** SinglePointOfFailure
- **User Journey:** journey-2
- **Description:** The entire SaaS platform is deployed in a single region (us-west-2) without cross-region redundancy. If the us-west-2 region experiences an outage, all user journeys (Browse Products, Place Orders, Manage Products, Onboard Tenants, View Orders) become completely unavailable. This affects all tenants simultaneously, causing total business disruption including revenue loss from order placement failures and customer acquisition impact from tenant onboarding failures.

---

## Medium Severity Findings (4)

### 4. API Gateway VPC Link lacks redundancy for EKS connectivity
- **Finding ID:** 1079af1f-cd25-4c52-8703-04838fce21b8
- **Severity:** Medium
- **Status:** Open
- **Failure Category:** SinglePointOfFailure
- **User Journey:** journey-4
- **Description:** The API Gateway uses a single VPC Link (ekssaasvpclink3DA7D40E) to connect to the EKS cluster. If this VPC Link fails or becomes unavailable, all API requests to the microservices will fail, causing complete outage of the Onboard Tenants journey and preventing tenant management operations through the control plane API.

### 5. Missing Pod Disruption Budgets allow unsafe pod evictions during maintenance
- **Finding ID:** 297fdde0-3a7b-44d5-a2ae-837fa905e107
- **Severity:** Medium
- **Status:** Open
- **Failure Category:** MisconfigurationAndBugs
- **User Journey:** journey-2
- **Description:** The EKS cluster lacks Pod Disruption Budget (PDB) resources for the product and order services. During cluster maintenance, node upgrades, or voluntary disruptions, Kubernetes may evict all pods simultaneously, causing complete service outages. This particularly impacts the Place Orders journey during maintenance windows, potentially causing revenue loss if customers cannot complete purchases.

### 6. EKS nodegroup minimum size of 1 creates potential compute SPOF
- **Finding ID:** 7416cec2-3d63-41df-897a-78d11eea46cc
- **Severity:** Medium
- **Status:** Open
- **Failure Category:** SinglePointOfFailure
- **User Journey:** journey-1
- **Description:** The EKS nodegroup is configured with MinSize: 1, meaning the cluster could scale down to a single node. If that single node fails or becomes unavailable, all pods (product and order services) would be unable to schedule, causing complete failure of Browse Products, Place Orders, and all other user journeys until a replacement node is available.

### 7. DynamoDB tables use low provisioned capacity risking throttling under load
- **Finding ID:** cd58e7e4-29f3-49ce-a3fd-be229b9df0ec
- **Severity:** Medium
- **Status:** Open
- **Failure Category:** ExcessiveLoad
- **User Journey:** journey-4
- **Description:** All DynamoDB tables (ProductsTable, TenantTable, TenantDetails) are configured with only 5 RCU/WCU each. During peak usage or tenant onboarding spikes, these tables will throttle requests, causing the Onboard Tenants journey to fail when creating new tenant records, and impacting Browse Products and Place Orders journeys when accessing product or tenant data under load.
