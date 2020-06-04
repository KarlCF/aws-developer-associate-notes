## Route 53

* Most common records are:
  * A: URL to IPv4
  * AAAA: URL to IPv6
  * CNAME: URL to URL
  * Alias: URL to AWS resource
* Route53 can use:
  * Public domain names you own
  * Private domain names that can be resolved by your VPCs. (ex: application1.company.internal)
* Route53 has advanced features such as:
  * Load Balancing (Through DNS - also called client load balancing)
  * Health checks (although limited...)
  * Routing policy: simple, failover, geolocation, geoproximity, latency, weighted
* Prefer Alias over CNAME for AWS resources (bettter performance)
