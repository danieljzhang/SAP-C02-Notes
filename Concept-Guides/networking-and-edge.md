# Networking and Edge

## Choose the connection by what must be reachable

Start with the communication pattern: a whole network, a single service, a remote user, or a public application.

- **VPC peering** connects two VPCs directly, but peering is not transitive. It requires non-overlapping address ranges for routed communication.
- **AWS Transit Gateway** provides a regional hub for connecting many VPCs and network attachments. Its route tables can segment traffic, but attachments and routes still need deliberate configuration.
- **AWS PrivateLink** exposes a specific service through a private endpoint rather than routing between the consumer and provider VPCs. Consumers create interface endpoints with private IP addresses. A provider publishes an endpoint service, commonly fronted by an NLB; Gateway Load Balancer endpoint services support network appliances. Because the consumer is not routing into the provider VPC, this pattern can work when VPC address ranges overlap.
- A **gateway endpoint** provides private access to supported services such as S3 and DynamoDB through route-table entries. An **interface endpoint** creates endpoint network interfaces and is used for many other AWS services and PrivateLink services.

An endpoint is not an automatic permission grant. Combine network reachability with security groups, endpoint policies where applicable, and the destination service's authorization rules.

**Fresh example:** Two acquired companies have overlapping VPC address ranges, but one needs to call a catalog API owned by the other. Expose only that API through a PrivateLink endpoint service instead of trying to route all traffic between the VPCs.

## Hybrid connectivity

- **AWS Direct Connect (DX)** provides a dedicated network connection to AWS. A private virtual interface is used for private connectivity to VPC-related gateways; a public virtual interface reaches supported AWS public endpoints; a transit virtual interface connects through a Direct Connect gateway to a Transit Gateway.
- A **Direct Connect gateway** helps associate a DX connection with supported virtual private or transit gateways. It is not itself a general-purpose router between every attached network.
- DX is not inherently encrypted. If traffic needs encryption in transit, an IPsec Site-to-Site VPN can be layered over the connection where supported. Use redundant connections and independent failure paths when availability requirements demand them.
- **Site-to-Site VPN** joins customer networks to AWS over IPsec tunnels. **AWS Client VPN** is for individual remote users connecting to VPC resources; it is not the same as a network-to-network VPN.
- A **forward proxy** makes or filters application-layer outbound requests on behalf of clients. A NAT gateway translates addresses for outbound connections but does not provide equivalent user-aware web filtering or request-level controls.

## DNS and traffic selection

Amazon Route 53 supports public and private hosted zones, health checks, and routing policies. Choose a policy based on the intent: weighted distribution for controlled proportions, latency for low-latency regional selection, geolocation/geoproximity for location-based rules, failover for primary/secondary behavior, and multivalue answers for returning multiple healthy records. DNS answers are cached by clients and resolvers; DNS routing is not a substitute for an application load balancer.

A Route 53 **Alias** record can target supported AWS resources and can be used at a zone apex. A DNS CNAME cannot be placed at the zone apex. Use ordinary A/AAAA records for literal IP addresses and CNAMEs for supported name-to-name mappings.

For hybrid DNS, Route 53 Resolver **inbound endpoints** let networks outside a VPC query names resolved in AWS. **Outbound endpoints and forwarding rules** let VPC workloads send selected DNS queries to external DNS resolvers. To use a private hosted zone across accounts, arrange the hosted-zone/VPC association and any required RAM sharing or cross-account authorization steps.

DNSSEC protects DNS data integrity and authenticity; it does not encrypt DNS queries. Validate signing, key management, and registrar/parent-zone requirements as a complete chain.

## Content delivery, TLS, and global entry points

**CloudFront** caches and delivers content through edge locations. It can use S3 or a custom origin, including an origin that is reachable over the network. Treat TLS as two connections: viewer-to-CloudFront and CloudFront-to-origin. Configure certificates and origin protocol policies for both legs; a secure viewer connection alone does not prove the origin connection is encrypted.

For a certificate managed by ACM and attached to a CloudFront distribution, request or import that certificate in US East (N. Virginia) (`us-east-1`). ACM certificates used by other regional services generally need to exist in the service's Region.

CloudFront **AWS WAF** rules filter HTTP(S) requests, while **geo restriction** limits delivery by viewer location. Neither feature replaces application authorization. **Field-level encryption** protects selected request fields so only an authorized downstream component can decrypt them; it is distinct from ordinary TLS.

**Lambda@Edge** and **CloudFront Functions** let you customize selected viewer or origin request/response events. They have different runtime, event, and feature capabilities; use the lightest option that meets the requirement and verify its current restrictions before deployment.

A CloudFront 504 points to an origin response or connectivity problem. Check origin health, name resolution, network/security rules, response timeouts, and origin capacity. Origin failover is for configured failure conditions, not a general fix for a slow or unhealthy origin.

For API responses, choose the cache at the layer that understands the request and desired cache key. API Gateway caching is API-stage behavior; CloudFront is useful for broadly cacheable edge responses; DynamoDB Accelerator (DAX) is a cache for eligible DynamoDB read paths. Private or user-specific responses need careful cache-key and authorization design.

**AWS Global Accelerator** provides static anycast entry IPs and steers supported TCP/UDP traffic toward healthy regional endpoints. It can complement CloudFront, but it does not cache or transform web content. Favor CloudFront for HTTP content delivery and caching; consider Global Accelerator when stable IPs, rapid regional failover, or non-HTTP transport matter.

## Managed transfers and local infrastructure

AWS Transfer Family provides managed file-transfer endpoints for supported protocols and storage backends such as S3 or EFS. Decide whether the endpoint should be publicly reachable or privately accessible through a VPC, then restrict network access and identities accordingly.

AWS Outposts places supported AWS infrastructure in a customer location for workloads with local processing, residency, or low-latency needs. It remains integrated with an AWS Region and has capacity, service-availability, and connectivity dependencies. Distinguish this infrastructure extension from Greengrass, which runs edge software on customer-managed devices.

## References

- [Amazon VPC connectivity options](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html)
- [AWS PrivateLink concepts](https://docs.aws.amazon.com/vpc/latest/privatelink/concepts.html)
- [AWS Transit Gateway](https://docs.aws.amazon.com/vpc/latest/tgw/what-is-transit-gateway.html)
- [AWS Direct Connect](https://docs.aws.amazon.com/directconnect/latest/UserGuide/Welcome.html)
- [Route 53 routing policies](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/routing-policy.html)
- [Route 53 Resolver for hybrid DNS](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/resolver.html)
- [CloudFront Developer Guide](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/Introduction.html)
- [Lambda@Edge](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/lambda-edge.html)
- [AWS Global Accelerator](https://docs.aws.amazon.com/global-accelerator/latest/dg/what-is-global-accelerator.html)
- [AWS Transfer Family](https://docs.aws.amazon.com/transfer/latest/userguide/what-is-aws-transfer-family.html)
