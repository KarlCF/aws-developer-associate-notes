## **Route 53**

* Managed DNS
* In AWS, most common records are:
  * A: hostname to IPv4
  * AAAA: hostname to IPv6
  * CNAME: hostname to hostname
  * Alias: hostname to AWS resource
* Route53 can use:
  * Public domain names you own or buy
  * Private domain names that can only be resolved within your VPC
* Route53 has advanced features as well, such:
  * Load Balancing (through DNS - also called client load balancing)
  * Health checks (although limited)
  * Routing policy: simple, failover, weighted, geolocation, latency, multi-value
* Billing is $0.50 per month per hosted zone

### DNS Records TTL

* **High TTL(e.g. 24h)**
  * Less DNS traffic
  * Possibly outdated records
* **Low TTL(e.g. 60s)**
  * Higher traffic on DNS
  * Records are outdated for less time
  * Easy to change records
* **TTL is mandatory for each DNS record**

### CNAME vs Alias

* AWS Resources (Load Balancer, CloudFront...) expose an AWS Hostname:
  * lbl-1234.us-east-2-elb.amazon.com and you want myapp.mydomain.com
* **CNAME**:
  * Points a hostname to any other hostname (app.mydomain.com => blabla.anything.com)
  * **ONLY FOR NON ROOT DOMAIN(e.g. something.domain.com)**
* **Alias**:
  * Points a hostname to an AWS Resource (app.domain.com => blablabla.amazonaws.com)
  * **Works for NON ROOT and ROOT domain**
  * Free of charge
  * Native health check

### Routing Policy - Simple

* Use when you need to redirect for a single resource
* You can't attach health checks to simple routing policy
* If multiple values are returned, a random one is chosen by the **client**

### Routing Policy - Weighted

* Control the % of the requests that go to specific endpoint
* Helpful to test 1% of traffic on new app for example
* Helpful to split traffic between two regions
* Can be associated with Health Checks

### Routing Policy - Latency

* 

### Route53 Health Checks

* 

### Routing Policy - Failover

* 

### Routing Policy - Geolocation

* 

### Routing Policy - Multi Value

* 