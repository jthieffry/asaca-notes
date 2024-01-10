# Key Notes

## CloudFront Architecture
* Caching system for data, to make it closer to your users. 
* Origin: source location of your content. 
* Can be either S3 Origin or Custom Origin (everything else). 
* Distribution: The "configuration" unit of CloudFront
* Behaviour: "Sub-configuration" unit of distribution. Adds important settings such as TTL, routing pattern, etc. 
* Edge Location: local cache of your data. Approx. 200 worldwide. Small, mostly storage. 
* Regional Edge Cache: Larger version of an edge location. Provides another layer of caching. 
* CloudFront integrates with ACM (certmanager) for https. 
* CAN ONLY BE USED FOR DOWNLOAD. Upload reqs go directly to origin. 

## Distributions and Behaviours
* Behaviours are the configuration unit. You set some important settings such as:
    - Which edge loc to use (all or only US/EU for ex)
    - An alternate dns name
    - Custom ssl cert and Security Policy (ex TLSv1.2_2018)
    - ...
* Distributions are sub configuration items that are evaluated against a path, with * being the default and the less selective one. 
* Distribution controls:
    - Protoco policy (http/s)
    - Allowed HTTP methods (ex only get, head or post)
    - VIEWER ACCESS RESTRICTION (trusted key groups)
    - CACHE SETTINGS

## TTL and Invalidation 
* More frequent cache hits = lower origin load (good)
* Default TTL (set at behaviour level): 24hrs (validity period)
* Can set minimum and max ttl values. 
* During a request (S3 via object metadata or custom), the client can set the following headers:
    - Origin Header: Cache-Control max-age and Cache-Control s-maxage (both in seconds)
    - Those explicitely sets what the client considers expired or not and will trigger a refresh when appropriate. 
    - Client can also set Origin Header: Expires (date & time)
* Cahce invalidations are performed on a DISTRIBUTION
* Applies to all edge locations in scope. It takes time. 
* Applies to a path. Ex. /images/img1.jpg or /images/*. 
* If need to invalidate often and if money/perf is an issue, consider using versioned file names instead and update the app(ex img_v1.jpg, img_v2.jpg). 

## ACM
* Lets you run a public or private CA. In case of the latter, your apps need to trust it. 
* Can also generate or import certs. 
* If generated, ACM can automatically renew them. 
* If imported, you are responsible for the renewal. 
* Certs can be automatically deployed out to supproted services. 
* Supported AWS services ONLY (Cloudfront and Alb. EC2 NOT SUPPORTED). 
* ACM is a regional svc. 
* CERTS CANNOT LEAVE THE REGION THEY ARE IMPORTED/GENERATED FROM. 
* To use a cert with an alb in region-a you need a cert in acm in region-a. 
* Global svc such as cloudfront operates as though within region us-east-1. 

## Cloudfront and SSL
* Cloudfront default domain name has the form <randombits>.cloudfront.net. 
* For that one, ssl is supported by default via the wildcard *.cloudfront.net cert. 
* For alternate domain name (ex. cdn.myapp.com), cert need to be generated or imported in ACM in region us-east-1 (since cloudfront is a global svc). 
* From a conn perspective there is actually two conn:
    1. Between viewer (client) to cloudfromt (viewer protocol)
    2. Between cloudfront to origin (origin protocol)
* BOTH CONNS NEED VALID PUBLIC CERTS (not private). 
* If client uses old browser that doesnt support sni, need to provide dedicated ip for them at each edge location for ssl. Approx 600$ per month. 
* For the origin protocol, origins need to have certs issued by a trusted ca. Alb can use acm, others need to use an external generated cert. NO SELF SIGNED CERTS. 
* If origin is s3, s3 handles this natively so no extra config needed. 

## Cloudfront origin types
* Either S3, Media Package, Media Store, Other (custom origins - webservers)
* Whats available in term of options depends of ghe origin type. 
* For S3 origin, you can restrict the access to cloudfront only via option origin access. 
* You can also setup the automatic bucket policy update to allow read access. 
* If using custom origin you can:
    - Decide on the origin protocol (http only, https only or match viewer)
    - Select the http port of the origin (default 80). 
    - Select the minimum origin ssl proto (select the latest one supported by the default origin as a safe default). 
    - Can add custom headers understandable by the origin in order to for example restrict origin access for cloudfront only. 

## Securing Origin Path to ensure it is coming from CF
* Origin Access Identity (OAI):
    - It is a type of identity similar to a role that can be associated with a CF distribution. 
    - CF then 'becomes' that OAI
    - That OAI can then be used in S3 bucket policies
    - Something like DENY All BUT one or more OAI
    - OAI ARE ONLY FOR S3 origins. 
* And what about custom origins security then?
    - Enforce https for the viewer and origin proto 
    - Inject custom headers at the CF distribution level and require those headers at the app level. 
    - Or, since aws publishes the list of public ips of his CF edge locs, just put a FW around your custom origin and whitelist those ips. 

## Securing viewer protocol
* A distribution is either public or private
* Public: open access to object, private: requests require signed cookie or url
* If a distribution has only one behaviour: the whole distribution is either public or private (depending on the behaviour)
* If it has multiple behaviors: each is public or private. 
* Regarding private distributions:
    - The old way is to create a CF key by an account root user. This account is then added as a trusted signer. 
    - The new way leverages trusted key groups instead. 
* Signed URls:
    - Provides access to one object only. 
    - Use urls if your client doesnt support cookies. 
* Cookies:
    - Provide access to a group of objects. Use for group of file or all files of a type. 
    - Use if maintaining the application url is important. 

## Lambda@Edge
* Lightweight lambda at edge location
* Adjust data between viewer and origin
* Only supports nodejs and python for now
* Only runs in aws public space (not vpc)
* Layers are not supported
* Different limits wrt normal lambda functions
* Lambdas can be set at:
    - The viewer request phase
    - The origin request phase
    - The viewer response phase
    - The origin response phase
* Example scenarios for lambda on edge includes:
    - AB testing (viewer request)
    - Migration between s3 origins (origin request)
    - Different obj based on device (origin request)
    - Content by country (origin request)

## Global Accelerator
* Moves the aws network closer to customer
* Connection enter at edge using anycast ip
* Transit over aws backbone to 1+ location 
* Can be used for non http (tpc/udp). Cannot do caching. Differrnce from CF. 
