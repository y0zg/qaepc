# Manifest

- K8s native:
```
kubectl apply -f manifests
```

- Helm 3
```
helm install nginx nginx-helmchart
```

# Multiregion

Whitepaper: https://docs.aws.amazon.com/whitepapers/latest/building-scalable-secure-multi-vpc-network-infrastructure/vpc-to-vpc-connectivity.html

## Option 1: VPC peering
- Benefits:
  - Simplest solution to connect 2 VPCs.
  - Lowest overall price
- Disadvantages:
  - Peer-to-peer design
  - Doesn’t support transitive routing and is difficult to scale. There is a max limit of 125 peering connections per VPC.
  - On-premise connectivity (VPN/DirectConnect) must be made to each VPC.

## Option 2: Transit VPC
- Benefits:
  - Hub and spoke design, more flexible than VPC peering
- Disadvantages:
  - Higher cots comparing to VPC peering
  - Limited throughput per VPC (up to 1.25 Gbps per VPN tunnel)
  - Customers have to manage HA
  - Only 1 hub VPC is possible
  - VPNs are used under the hood which gives additional overhead if performance is critical
	
Reference: 
https://docs.aws.amazon.com/whitepapers/latest/building-scalable-secure-multi-vpc-network-infrastructure/transit-vpc-solution.html

## Option 3: PrivateLink
- Benefits:
  - AWS PrivateLink endpoints can be accessed over VPC Peering, VPN, and AWS Direct Connect
- Disadvantages:
  - Client/server design. Doesn’t support bidirectional access

## Option 4: AWS Transit Gateway (preferred solution if costs are not a problem)
- Benefits: 
  - Hub and spoke design
  - High availability out of the box
  - Allows to connect non-AWS and on-premise networks
  - It will be possible to extend AWS TGW route domains to create segment VPC workloads and define connection policies across  VPCs (Dev, Prod, QA)
  - Integration and centralized accounts managements via AWS Control Tower
- Disadvantages:
  - Cost, extra $0.05/hour per VPC connection comparing to VPC peering

Reference: 
https://docs.aws.amazon.com/whitepapers/latest/building-scalable-secure-multi-vpc-network-infrastructure/transit-gateway-vs-transit-vpc.html
https://docs.aws.amazon.com/whitepapers/latest/building-scalable-secure-multi-vpc-network-infrastructure/transit-gateway-vs-vpc-peering.html

## Option 5: ServiceMesh (not a production ready solution)
Istio Multi-cloud Service Mesh supports virtual machines

Reference:
https://istio.io/latest/docs/examples/virtual-machines/single-network/#send-requests-from-virtual-machine-workloads-to-kubernetes-services

## Option 6: Connectivity on application level

Let's assume we have a legacy application running on EC2 instances and decided to migrate it to EKS. In this case leveraging AWS EventBridge + SQS might be helpful as an interim solution if it’s not possible to use the “lift and shift” strategy. So, Legacy application will be sending events to EventBridge  with routing to SQS queue. Simple SQS consumer client (AWS SDK) will be deployed in EKS.

# Database

Based on the description it’s not clear that the database is really a bottleneck, so proper monitoring, logging and tracing should be in place.

Recommendations: 
- Since we deal with the payment system, all transactions must obey ACID properties. 
- Considering that consistency and availability are important, we can refer to CAP theorem and use any relational database which suits our needs.
- Use dedicated database for payments (shouldn’t be shared with other services/domains)
- Use asynchronous payments and leverage message queue for fault-tolerance.
- Considering we use AWS and high availability is important , we can use Aurora multi-master cluster mode.. 
- RDS proxy feature could increase HA further