## Chapter1. Microservice Security Landscape
### 1.1 How security works in a monolithic application
- The servlet filters authenticates the request, and adds user information in the web session.
- The inner components assume that the request is valid.
- The security is controlled centrally

### 1.2 Challenges of securing microservices
  - More entry point, more points to be protected
  	- mono: one entry point, in-process call on one jvm
  	- micro: many entry points(each one has its own). in-process call -> remote call
  - distributed security screening -> poor performance
    - monolithic: security check once
    - mociroservice: repetitive check(w/ remote security token service) \
    -> redundant checks, performance issue
   - deployment complexities
     - each microservice need certifiacte which is used in authentication. Need way to revoke and rotate certificates
  - hard to trace the request
    - three pillars of observability: log, metrics, traces
    - request may go through multiple microservices before leaving the system since it enters.
  - conatiners are immutable, but allowed clients are dynamically decided.
    - store access-control policy in different server(the policy administration end-point)
      - push model: the policy administration end-point pushes policy updates to the interstes mocioservices
      - pull model: each microservice polls the policy administration end-point
    - certificate should be rotated, injected when the service boots up
  - sharing user context is hard since distributed
  - polyglot architecture -> each one use different tech stack

### 1.3 Key Security fundamentals
- Authentication: identify the request party
  - clarify the audience first
  - system delegates human user: OAuth 2.0
  - authenticate human user: MFA(Multi Factor Authentication) ex) OTP, FIDO(Fast Identity Online)
  - authenticate system: certificates, JWT
- Integrity: check if the data is not modified
  - data in transit: sing-in. ex) TLS, https
  - data in rest: audit trailing, periodically calculate message digest of audit trail, encyrpt, and store securely
- nonrepundiation: the microservice can not deny that he didn't do the transaction
  - digital signature, record of signature and timestamp
- confidentiality: protect from uninteded disclosure
  - when hijacked, data should not be understandable
  - encryption. However, it is expensive
  - data in transmit: ex) TLS \
    with proxy server,
    - TLS bridging: proxy server terminates TLS connection and creates a new one. Data is readable in the proxy server.
    - TLS tunneling: proxy server creates a tunnel. Data is unreadable in the proxy server.
  - data in rest: database
- availiability: the system should be running, no matter what
  - Security takes a key role in making system constantly available to stakeholders
  - firewall, api gateway
- authorization: do you have a permission?

### 1.4 Edge Security
- role of API Gateway
  - expose a selected set of microservices to the outside world as API
  - build quality-of-service(QoS) features(security, thorttling, analytics)
- authentication
  - certificate-based: server to server. mTLS
  - Oauth 2.0: human(delegate to server) to system
- authorization
- passing client/user context to upstreaam microservices: http header, jwt

### 1.5 Securing service-to-service communication
- service-to-service communication
  - synchornous: http
  - asynchronous: message queue
- service-to-service authentication
  - trust the network
    - old-school model
    - no security
    - opposite of zero-trust network(every request is checked)
  - mTLS
    - transport layer
    - each microservice carries public/private key
  - JWT
    - application layer
    - JWT includes claims and signed by the issuer. \
    issuer can be STS or the micrsoervice itself
    - works over TLS
- service-level auhtorization
  > PDP(Policty Decision Point)
  - centralized PDP model
    - access-control policies are defined, stored, and evaluated centrally
    - to authenticate, need to talk to the endpoint
    - create dependency on the PDP and increase latency \
    caching can decrease latency
  - embedded PDP model
    - policies defined centrally but stored and evaluted at the service level
    - the challenge: how to update policy from the centralized policy administration point(PAP)
      - poll or push
      - since servers load policies when boot, the service should restart to reload. -> overkill
- propagating user context among microservices
  - http header: can be modified
  - JWT issued by calling service: calling service can lie th euser context
  - JWT issued by external STS: most secure
- crossing trust boundaries
  - w/o api gateway between trust domain
    1. api gateway -> order processing service with jwt signed by the gateway, aud: order processing service
    2. order processing microservice -> STS of foo domain.
    3. foo STS returns new JWT signed by it, aud: STS of bar
    4. and 5. order processing microservice -> STS of bar, get new JWT signed by STS of bar, aud: delivery microservice
    6. order processing microservice access declivery microservice
    ![without gateway](without_gateway.jpg)

  - w api gateway between trust domain
    1. api gateway -> order processing service with jwt signed by the gateway, aud: order processing service
    2. order processing microservice -> STS of foo domain.
    3. foo STS returns new JWT signed by it, aud: apigateway of the bar
    4. order processing microservice -> api gateway of bar
    5. api gateway of bar -> STS of bar \
    bar STS creates jwt with aud: delivery microservice
    6. api gateway of bar -> delivery microservice
    ![with gateway](with_gateway.jpg).

## Chpater2. Fist steps in securing microservice
![Oauth](oauth.jpg).

## Chapter3. Securing north/south traffic with an API gateway

### 3.1 The need for an api gateway in a microservice deployment
- Decoupling security from the microservice
  - Change of security protocol require changes in the microservice
  - Scaling up the microservice results in more connecdtions to the authorization server
- Consistent interface while microservices may be inconsistent. \
  (microservices tend to have diverse tech stack, complex structures)
- hide inner structure
  - in some cases, read happens much more than writes
  - with api gateway, we can divide read service and write service but maintain the identical entrypoint.

### 3.2 Security at the edge(why OAuth 2.0?)
- diverse consumers: internal, external, hybrid applications
- delegating access: client application access on behalf user
- why not basic authentication?
  - username and password are static, long-living
  - no scope check
- why not mTLS?
  - has expiration, which solves basic athentication's problem but...
  - no access delegation
- Why OAuth 2.0?
  - who: only permitted entities
  - what purpose: scope
  - how long: ensure that access is granted for desired period

### 3.3 Setting up an API gateway with Zuul
- Firewall blocks the direct access from client application to microservice/authorization server
- JWT
  - Authroization server is hard to scale but, recieves many requests.
  - If access token contains information needed for authentication, we can solve this problem.
  - pitfalls of self-validating token
    - no way to revoke prematurely
      - solution1: short-lived JWT
      - solution2: inform the API gateway of the revoked tokens
    - certificate used to verify a token signature might have expired. \
    then, JWT can no longer be verified.
      - solution1. deploy the new certificate when renewed
      - solution2. provision the ceritifacte of the CA(certificate authority) of the token issuer.

### 3.3 Setting up an API gateway with Zuul
- preveneting access with the firewall
- use mTLS in communcation between the api gateway and microservices


## Chapter4. Accessing a secured microservice via a single-page application

### Architecture of SPA
![spa architecture](spa_arch.jpg)

### Authentication flow of SPA
- authentication flow

  ![spa authentication flow](spa_auth_flow.jpg)

- with webserver

  ![spa with web server](spa_web_server.jpg)

- with resource server

  ![spa with resource server](spa_resource_server.jpg)

### With single/multi trust domains
- The client application(SPA), authorization server, and resource server need to be in the same trust domain.

  ![spa with with single trust domain](spa_single_trust_domain.jpg)
- To build trusts between multiple trust domains,
  - web server should exchange token from authorizations to the one for domain2.
  - in case of JWT, JWT bearer grant type accepts a JWT, validates it, and issues a valid OAuth 2.0 token(whic is another JWT or opaque token string)
  ![spa with with multi trust domains](spa_multi_trust_domains.jpg)


## Chapter5. Engaging throttling, monitoring, and access control
> Throttling \
> a. the suppression or prevention of an activity \
> b. the activity or process of limiting the bandwidth available to users of an electronic communication systems (such as the Internet) \
> referenced: merriam-webster dictionary

### 5.1 Throttling at the API gateway with Zuul
- scaling is not free. \
Also, there is a physical limit on how much we can scale the system. \
Thus limiting the number of requests(throttling) is needed.
- basic notion: count the number of requests and if exceeds the limit, block
  - Quota-based
    - provide each application a quota
    - identify application with client-id which can be retrieved from auth server with access token
  - fair usage policy for user
    - in case of multiple users per application, ex) 20 users on one application
    - ex) JWT, use sub claim to identify user
- set maximum threshold limit
- operation level throttling
  - Single api may have two microservices. One for read, the other for write.
  - Throttling may be required on operation-level
- Throttling the OAuth 2.0 token and authorize endpoints
  - Throttling on auth server api is hard, since the requests are preauthenticated, which means we can identify the source of the request
  - Only ip is available, and it is out of the handling scope of the api gateway. Instead, deal with firewall
- privilege-based throttling
  - highger priviliege may have more quota

### 5.2 Monitoring and analytics with Prometheus and Grafana
- prometheus pulls metrics data from microservice periodically
- Also, it has a push-gateway for short-lived process, to send-data before prometheus polls

![prometheus](./prometheus.jpg)

### 5.3 Enforcing access-control policies at the API gateway with Open Policy Agent
![opa](./opa.jpg)



## Chapter6. Securing east/west traffic with certificates
### 6.1 Why use mTLS?
- TLS
  - with tls, no one in the middle can see what it is
  - client application can identify the server that it communicates with
  - the server should have valid certificate that is trusted by all the client applications
- CA(certificate authority)
  - A third party that signs the certificate given to the service
  - Few trusted CA exist, their public keys are embedded in all browsers
  - browser can verify that the service's certificate is valid by verifying the signature against the CA's public key.
- mTLS
  - both server and client msut have valid certificate and trust corresponding certificates.
  - secure communications between microservices

### 6.5 Challenges in key management
- bootstraping trust
  - have a single CA, and each microservice trust this CA
- provisioning key/certificates to workloads or microservices
  - typical key-provisioning process at an enterprice
    - generate public/private key pair
    - create a certificate signing request(CSR) & submit
    - when acepted, deploy the certificate and the keys to the microservice
  - at netflix
    - short-lived credential
  - generating long-lived credentials
    - netf
  - SPIFFE
    - SPIFFE(Secure Production Identity Framework For Everyone): open standard that defines a way a microservice can establish an identity
    - SPIFFE Runtime Environment: open source reference implementation of SPIFFE
- key revocation
  - cases
    - private key is compromised
    - CA's private key is compromised
    - the detail of CSR is invalid
    - certificate does not represent the original entity at the time the certificate was issued.
  - CRL(certificate revocation list): old way, up to TLS client's application
  - OCSP(Online Certificate Status Protocol)
    - tls client asks to the OCSP end point if the certificate is revoked, every time it sees.
    - increase client-side latency
    - unclear if the response contain updated revocation information. cached response
    - response can be faked, filtered, etc.
    - privacy risk. OCSP responder knows end user
  - OCSP stapling
    - server recieves the staple from the OCSP end point and attach it when it responds to the client
  - short-lived certificate
    - only rely on expiration
    - short-lived certificate is in-memory thus cheap, \
    whereas loading long-lived certificate is expensive
    - netflix uses a layered approach \
      during bootup, use long-lived credentials, then each microservice get short-lived credentials using the long-lived credentials
      ![netflix](./netflix.jpg)

- key rotation
- monitoring key usage