## Security - KMS

### Why encryption?

#### Encryption in flight (SSL)

* Data encrypted before sending and decrypted after receiving
* SSL certificates help with encryption (HTTPS)
* Encryption in flight ensures no MITM (man in the middle attack) can happen

#### Server side encryption at rest

* Data is encrypted after being received by the server
* Data is decrypted before being sent
* It is stored in an encrypted form thanks to a key (usually a data key)
* The encryption / decryption keys must be managted somewhere and the server must have access to it

#### Client side encryption

* Data is encrypted by the client and never decrypted by the server
* Data will be decryted by a receiving client
* The server should not be able to decrypt the data
* Could leverage Envelope Encryption

### AWS KMS (Key Management Service)

* Easy way to control access to your data, AWS manages keys for us
* Fully integrated with IAM for authorization
* Seamlessly integrated into various AWS services
* Also usable with CLI / SDK

#### KMS - CMK (Customer Master Keys) Types

* **Symmetric (AES-256 Keys)**
  * First offering of KMS, single encryption key that is used to Encrypt and Decrypt
  * AWS services that are integrated with KMS use Symmetric CMKs
  * Necessary for envelope encryption
  * You never get access to the Key unencrypted (must call KMS API to use)
* **Assymetric (RSA & ECC key pairs)**
  * Public (encrypt) and Private Key (Decrypt) pair
  * Used for Encrypt/Decrypt, or Sign/Verify operations
  * The public key is downloadable, but you access the Private Key unencrypted
  * Use case: encryption outside of AWS by users who can't call the KMS API

#### AWS KMS CMK - Symmetric Keys

* Able to fully manage keys & policies:
  * Create
  * Rotation policies
  * Disable
  * Enable
* Able to audit key usage (CloudTrail)
* Three types of CMK:
  * AWS Managed Service Default CMK: **free**
  * User Keys created in KMS: 1$ / month
  * User Keys imported (must be 256-bit symmetric key): 1$ / month
* Pay for API calls to KMS ($0.03 / 10000 calls)

#### AWS KMS 101

* Use KMS anytime you need to share sensitive information:
  * Database passwords, Credentials to external service, Private Key of SSL certificates, etc...
* The value in KMS is that the CMK used to encrypt data can never be retrieved by the user, and the CMK can be rotated for extra security
* **Never store your secrets in plaintext, especially in your code**
* Encrypted secrets can be stored in the code / environment variables
* **KMS can only help in encrypting up to 4KB of data per call**
* If data > 4KB, use envelope encryption
* To give access to KMS to someone:
  * Make sure the Key Policy allows the user
  * Make sure the IAM Policy allows the API calls
* KMS Keys are bound to it's region

#### KMS Key Policies

* Control access to KMS keys, "similar" to S3 bucket policies
* Difference: you cannot control access without them
* **Default KMS Key Policy**
  * Created if you don't provide a specific KMS Key Policy
  * **Complete access to the key to the root user = entire AWS account**
  * Gives access to the IAM policies to the KMS Key
* **Custom KMS Key Policy**
  * Define users, roles that can access the KMS key 
  * Define who can administer the key
  * Useful for cross-account access of your KMS key 

##### Copying Snapshots accross accounts

1. Create a Snapshot, encrypted with CMK
2. Attach a KMS Key Policy to authorize cross-account access
3. Share the encrypted snapshot
4. (in target account) Create copy of the Snapshot, encrypt it with a KMS Key in target account
5. Create a volume from the Snapshot

### Envelope Encryption

* KMS Encrypt API call has a limit of 4 KB
* Anything Over 4 KB of data that needs to be encrypted must use the Envelope Encryption == **GenerateDataKey API**
  * Generates a plaintext Data Key (DEK), then the key is used to encrypt the bigger file.
  * After the big file is encrypted, the encrypted Data Key is placed in an Envelope with the file (Envelope Encryption)
  * To decrypt the file, call the Decrypt API to decrypt the Data Key, and only then, decrypt the big file with the unencrypted Data Key

### Encryption SDK

* The AWS Encryption SDK implemented Envelope Encryption for us
* The Encryption SDK also exists as an CLI tool we can install
* Implementations for Java, Python, C, JavaScript
* **Feature - Data Key Caching**:
  * Re-uses data keys instead of creating new ones for each encryption
  * Helps with reducing the number of calls to KMS with a security trade-off
  * Use LocalCryptoMaterialsCache (max age, max bytes, max number of messages)

### KMS Symmetric - API Summary

* **Encrypt**: encrypt up to 4 KB of data through KMS
* **GenerateDataKey**: generates a unique symmetric data key (DEK)
  * Returns a plaintext copy of the data key
  * AND a copy that is encrypted under the CMK that you specify
* **GenerateDataKeyWithoutPlainxText**:
  * Generate a DEK to use at some point (not immediately)
  * DEK that is encrypted under the CMK that you specify (must use Decrypt later)
* **Decrypt**: decrypt up to 4 KB of data (including Data Encryption Keys)
* **GenerateRandom**: Returns a random byte string

### KMS Limits

#### KMS Request Quotas

* When you exceed a request quota, you get a **ThrottlingException**
* To respond, use **exponential backoff** (backoff and retry)
* For cryptographic operations, they share a quota
* This includes requests made by AWS on your behalf (ex: SSE-KMS)
* For GenerateDataKey, coinsider using DEK caching from the encryption SDK
* **You can request a Request Quotas increase through API or AWS support**
* **Symmetric CMK quota**:
  * 5.500 (shared)
  * 10.000 (shared): us-east-2, ap-southeast-1, ap-southeast-2, ap-northeast-1, eu-central-1, eu-west-2
  * 30.000 (shared): us-east-1, us-west-2, us-west-1
* **Asymmetric CMK quota**:
  * 500 (shared) for RSA CMKs
  * 300 (shared) for Elliptic curve (ECC) CMKs

