# Remaining ResilienceHub Findings - Architectural Changes Required

## Finding #3: Single-region deployment creates regional SPOF for entire SaaS platform
**Severity:** High  
**Status:** Requires Architectural Decision

### Issue
The entire SaaS platform is deployed in a single region (us-west-2) without cross-region redundancy. If the us-west-2 region experiences an outage, all user journeys become completely unavailable.

### Recommended Solution
Implementing multi-region deployment requires significant architectural changes:
- Deploy infrastructure to multiple AWS regions (e.g., us-west-2 and us-east-1)
- Implement cross-region DynamoDB global tables for data replication
- Set up Route53 health checks and failover routing
- Configure cross-region EKS cluster federation or active-active setup
- Implement application-level data synchronization

### Implementation Complexity
This is a major architectural change that requires:
- Business decision on RTO/RPO requirements
- Cost analysis for multi-region deployment
- Application code changes for multi-region awareness
- Testing and validation of failover scenarios

**Recommendation:** Defer to architectural review and business requirements discussion.

---

## Finding #4: API Gateway VPC Link lacks redundancy for EKS connectivity
**Severity:** Medium  
**Status:** Requires AWS Service Enhancement

### Issue
The API Gateway uses a single VPC Link (ekssaasvpclink3DA7D40E) to connect to the EKS cluster. If this VPC Link fails, all API requests to microservices will fail.

### Current Limitation
AWS API Gateway VPC Links are regional resources that are inherently highly available within a region. However, the CDK/CloudFormation does not currently support creating multiple VPC Links for redundancy at the application level.

### Recommended Solution
VPC Links are already deployed across multiple Availability Zones by AWS. The perceived "single point of failure" is mitigated by:
1. AWS's internal redundancy for VPC Link infrastructure
2. The Network Load Balancer behind the VPC Link is already multi-AZ
3. Our fix to ensure EKS nodes are distributed across AZs

### Additional Mitigation
If additional redundancy is required:
- Consider using AWS App Mesh for service mesh capabilities
- Implement circuit breakers and retry logic in application code
- Monitor VPC Link health metrics and set up alarms

**Recommendation:** Accept current AWS-managed redundancy; monitor and alert on VPC Link health.

---

## Summary of Fixes Implemented

### ✅ Fixed (5 findings)
1. **Finding #1 (High):** Increased pod replicas from 1 to 3 for product and order services
2. **Finding #2 (High):** Added topology spread constraints for AZ distribution
3. **Finding #5 (Medium):** Added Pod Disruption Budgets (minAvailable=2)
4. **Finding #6 (Medium):** Increased EKS nodegroup minSize from 1 to 2
5. **Finding #7 (Medium):** Changed DynamoDB to on-demand billing mode

### ⏸️ Deferred (2 findings)
1. **Finding #3 (High):** Multi-region deployment - requires architectural decision
2. **Finding #4 (Medium):** VPC Link redundancy - AWS-managed, monitoring recommended

## Next Steps

1. Deploy the fixes to AWS using `cdk deploy --all`
2. Run a new ResilienceHub V2 assessment to verify improvements
3. Schedule architectural review for multi-region deployment strategy
4. Set up CloudWatch alarms for VPC Link health monitoring
