## **AWS CloudFront**

* Content Delivery Network (CDN)
* Improves read performance, content is cached at the edge
* DDoS protection, integration with Shield, AWS Web Application Firewall
* Can expose external HTTPS and can talk to internal HTTPS backends

### CloudFront - Origins

* S3 Bucket
  * For distributing files and caching them at the edge
  * Enchanced security with CloudFront Origin Access Identity (OAI)
  * CloudFront can be used as an ingress (to upload files to S3)
* Custom Origin (HTTP)
  * Application Load Balancer
  * EC2 instance
  * S3 website (must first enable the bucket as a static S3 website)
  * Any HTTP backend you want

### CloudFront Caching

* Cache based on
  * Headers
  * Session Cookies
  * Query String Parameters
* The cache lives at each CloudFront Edge location
* You want to maximize the cache hit rate to minimize the requests on the origin
* Control the TTL (0 seconds to 1 year), can be set by the origin using the Cache Control Header, Expires header...
* You can invalidade part of the cache using the CreateInvalidation API

### CloudFront Security

#### CloudFront Geo Restriction

* You can restrict who can access your distribution
  * Whitelist: Allow your users to access your content only if they're in one of the countries on a list of approved countries
  * Blacklist: Prevent users from accessing your content if they're in one of the countries on a blacklist of banned countries
* The "Country" is determined using a 3rd party Geo-IP database

#### CloudFront and HTTPS 

* Viewer Protocol Policy:
  * Redirect HTTP to HTTPS
  * or HTTPS only
* Origin Protocol Policy (HTTP or S3):
  * HTTPS only
  * or Match Viewer (HTTP => HTTP & HTTPS => HTTPS)
* S3 bucket "websites" don't support HTTPS

### CloudFront Signed URL / Signed Cookies

* You want to distribute paid shared content to premium users over the world
* You can use CloudFront Signed URL / Cookie. We attach a policy with:
  * Includes URL expiration
  * Includes IP ranges to access the data from
  * Trusted signers (which AWS accounts can create signed URLs)
* How long should the URL be valid for?
  * Shared content (movie, music): make it short (few minutes)
  * Private content (private to the user): you can make it last for years
* **Signed URL** = access to individual files (one signed URL per file)
* **Signed Cookies** = access to multiple files (one signed cookie for many files)

#### CloudFront Signed URL vs S3 Pre-Signed URL

* CloudFront Signed URL:
  * Allow access to a path, no matter the origin
  * Account wide key-pair, only the root can manage it
  * Can filter by IP, path, date, expiration
  * Can leverage caching features
* S3 Pre-Signed URL:
  * Issue a request as the person who pre-signed the URL
  * Uses the IAM key of the signing IAM principal
  * Limited lifetime