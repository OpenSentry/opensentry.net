# Identity and Access Management System

  * [Vision](#vision)
  * [Why build a new system](#why-build-a-new-system)
    * [Issues of trust and control](#issues-of-trust-and-control)
    * [Free and transparent](#free-and-transparent)
    * [Minimal yet flexible](#minimal-yet-flexible)
    * [Secure by default](#secure-by-default)
    * [Dog food](#dog-food)
    * [Performant](#performant)
  * [Architecture for Minimum Viable Product](#architecture-for-minimum-viable-product)
    * [Dependencies](#dependencies)
    * [ORY Hydra](#ory-hydra)
  * [Concepts](#concepts)
    * [Identity](#identity)
    * [Role](#role)
    * [Challenge](#challenge)
    * [Scope](#scope)
    * [Publishing](#publishing)
    * [Subscription](#subscription)
    * [Grant](#grant)
    * [Consent](#consent)
    * [Shadow](#shadow)
    * [Rule](#rule)
  * [Wins](#wins)
    * [What does the system provide](#what-does-the-system-provide)
    * [OAuth2](#oauth2)
    * [Trust and Control](#trust-and-control)

## Vision
We want to provide an open source and free authentication, authorization and access management system. Useable as a micro services infrastructure foundation but not limited to that.

## Why build a new system
A myriad of systems providing identity authentication and authorization exist on the web and most are decently accessible, but like all system there is a price to pay either in the form of an actual price for enterprise features that often leave the open source variant of the software crippled and less than ideal, or there is a hidden cost in terms of architectural decisions making changes hard if not impossible.

### Issues of trust and control
Then there is the issue of trust and control. An identity and access provider is a logical single point of failure in any system architecture, effectively creating a logical barrier of protection if configured correctly, so what happens if it does not work? We took notice of a recent event where a certain president enforced an embargo, which left a region of business at a loss. What if that was to happen to your Identity and Access Provider? When choosing an identity and access provider one is effectively placing the trust of the operation of ones system into the hands of someone else.

A question we like to ask is: Would you give the key to your house front door to a door man, having to ask this door man every time you need access to your house? We would not.

Security is an exercise in trust and risk management and that is why control is of paramount importance.

In our search for a viable Identity & Access provider, all the solutions fell short in one or more aspects of what we want in a solution.

A list of ideas or philosophies we would strive towards for such a system:

### Free and transparent
The solution should be open source, free of charge and give you the freedom to change it to your needs.

Looking at ideas like Lets Encrypts bringing encryption to the masses and hence security for many more and effectively disrupting the business model of the certificate authorities, which do have their value when used correctly, but in a lot of cases just hinder encryption of the web by creating a pay wall. We wonder why is there no software bringing the same ideas to the most fundamental part of any system like authentication and authorization?

 Why is it, that a fundamental security element of systems required by most, like identity and authorization is not available to all in the same way? We would like to see that happen. Do not trust others to handle your system security take responsibility and take control.

### Minimal yet flexible
The solution should only contain bare minimum to grant the vision. It should not turn into a one size fits all feature fiesta. We need a way to authenticate an identity, a simple, not simplistic, way to authorize requests to execute functions on data on behalf of others.

We believe that less code and simple structures means better security and fewer bugs. We strive to adhere to the 12 factor methodology.

### Secure by default
Lots of identity and access provider systems are closed source, hosted in the cloud by others and leave us with a sense of inaccessible and hard to verify.

We want the system to have nothing to hide. No security by obscurity. Everything should be readable, even the password hashes if it was not for the fact that one should not trust the password hash to survive forever if leaked, hence we would not expose those in read calls, but anything else should be except if data is protected by requiring consent which the system should help enforce.

The system should not have any logic that does not survive the vigilant eyes of the knowing masses.

We want absence of information to mean no and denied, meaning everything stored in the system is data that gives never removes. We want the code fallback on error cases to always deny.

### Dog food
We are strong believers in the concept of dog food making software better, or sometimes known as bootstrapping a system with the system itself. We would like to use the system to secure the system. This gives a good boundary for the minimal viable product and leaving the system affected by errors affecting users alike. Hopefully giving better incentive to fix errors faster.

### Performant
A critical system like this should be as performant as possible and not just for the ones with enough gold. To achieve this we strive to be resource friendly.

For us this eliminates Java in its default configuration. Our hope falls on Go to deliver.

## Architecture for Minimum Viable Product
The system is composed of a set of services. For backend we have created two core components.

 - IDP - The Identity Provider
 - AAP - The Access & Authorization Provider

These are supplemented with 2 core frontend components.

 - IDPUI - The _trusted_ user interface for retrieving passwords from users
![Login UI](/images/login.png)
 - AAPUI - The _trusted_ user interface for retrieving consents from users
![Consent UI](/images/consent.png)

We have split the system like this to isolate and dog food interfaces between services, but also to exemplify which functions should not be shared lightly to third party access, if shared at all.

To showcase the functionality of IDP and AAP which is safe to delegate to 3. party untrusted applications, we created a frontend application that serves as the self service application exposing the actions that user expect from an identity and access provider.

  - MEUI - The _untrusted_ dashboard which showcase functionality of IDP and AAP safe for 3. party implementers

The first thing you'll see when entering MEUI is your profile page, where your data is listed with the abilities to enable 2FA, change password/email and edit your name.
![Showing your profile](/images/profile.png)

One of the more interessting places in MEUI, is "create a client". This will allow you to create your own apps with access to a resource servers data.
![Creating clients](/images/newclient.png)

In this example we have already created a resource server and published a test scope. Our new client can then be granted access to this.
![Give grants to others](/images/grants.png)

### Dependencies
To avoid having to implement a key safety feature of the system our selves. We rely on ORY Hydra to handle the Oauth2 protocol delegation for us. We also use other third party software to build parts of the infrastructure required for the system to operate.

 - ORY Hydra - An OAuth2 and OpenId Connect implementation
 - ORY OathKeeper - A reverse proxy implementation
 - NATS.io - An Event Messaging System
 - MySQL - One of the compatible storage engines for ORY Hydra data models
 - Neo4J - The graph database used to hold the data models of IDP and AAP
 - Docker - Containerization of the entire system

### ORY Hydra
We decided to use ORY Hydra as the OAuth2 delegator as the system is specifically designed to allow freedom in the implementation of login and consent process of Authorization Code flow.

They do this by using a login\_challenge directed at the login provider, and a consent\_challenge directed at the authorization provider with a straight forward redirect process. This separates them from other solutions on the web which often do not allow for implementation choices here.

## Concepts
A short introduction to concepts used by the IDP and AAP system.


### Identity
An Identity, owned by IDP, is a human, an application, a resource provider or anything that needs to be authenticated. An invite to the system is also an identity that upon confirmation gets converted to a human identity.
An Identity, owned by AAP, is anything that can publish, subscribe, consent and grant.

In the OAuth2 model this is the subject identifier.

### Role
A Role is an Identity, owned by IDP, that is not allowed to authenticate. It is used to augment an Identity for organizational purposes.

The OAuth2 model has no need for roles as they are an organization abstraction on subjects. They could be modeled with scopes if one wished it. We do not since organizational abstractions change all the time they should not be encoded into systems at a resource provider level.

### Challenge
A Challenge is an identifier owned by IDP, and used to authenticate an Identity or verify Identity control over a resource. Examples include confirm that an Identity has control over an email account or validate OTP passwords used for multi factor authentication.

### Scope
A Scope is owned by AAP and defines a textual interface of systems for use in OAuth2.

In the OAuth2 model this is simply a scope used to request access to resources at a resource provider.

### Publishing
A Publishing is owned by AAP and defined as an Identity publishing a set of functions and/or data to others. The granularity of the access control and the intended audience is entirely up to Identity creating the Publishing.

In the OAuth2 model this is an instantiation of a Scope as defined by a Resource Provider.

### Subscription
A Subscription is owned by AAP and defines a restriction on which Publishings are available to an Identity.

In the OAuth2 model a subscription is a limit on audience for clients effectively restricting which scopes a client can request access tokens for at resource providers.

Subscriptions has a simple design in our graph, which simply goes from a client to publishing. In this example, IDP is publishing the "idp:read:humans" scope, which our MEUI client is subscribed to. This allows MEUI to request the scope when talking to the OAuth2 Provider.
![MEUI subscription to IDP for reading a human](/images/subscription.png)

### Grant
A Grant, owned by AAP, is what gives identities access to resources. A Grant is defined as a connection between an Identity and a Publishing.

There are two main types of grant, the root grant and the grant on behalf of someone specific.

The following subset of the graph allows our "Test" user to read his own data. The "on behalf of" edge, tells us that this identity is only granted access to this specific data set. If this edge was pointing at the resource server itself (IDP), "Test" would have access to everyones data (used by client credientials):
![Allows identity to read own data](/images/grant-obo-self.png)

An example of a specific "client credientials" usecase, could be the ability to authenticate an identity (verify password). This lets our IDPUI client perform this for everyone, since we don't know who the user is, until we've authenticated the identity:
![Allows client to authenticate an identity](/images/grant-obo-rs.png)


### Consent
A Consent, owned by AAP, is what allow Identities to request scopes on behalf of an Identity. It is a connection between two Identities and a Publishing.

In the OAuth2 model this is a connection between a client requesting access to a resource provider on behalf of a subject.

The following subgraph shows how our "Test" user has consented to MEUI, giving the client access to read the data owned by "Test".
![Identity consented a client access to data](/images/consent-graph.png)

### Shadow
A Shadow, owned by AAP, is defined as a grant between two identities used to support Role Based Access Control, RBAC. This differs from Grant in that it does not grant a Publishing directly but an Identity instead which then might Grant publishings which will be inherited.

### Rule
A Rule is an instantiation of connected concepts owned by AAP. For instance a consent rule would specify who gave the consent? What was consented to? and to whom was it consented? Effectively connecting two identities to a publishing like a human given an application access to data on a resource provider.

 - PublishRule: (Identity)-[PUBLISH]->(PublishRule)-[PUBLISH]->(Scope)
 - SubscriptionRule: (Identity)-[SUBSCRIBE]->(SubscriptionRule)-[SUBSCRIBE]->(PublishRule)
 - ConsentRule: (Identity)-[CONSENT]->(ConsentRule)-[CONSENT]->(PublishRule), (ConsentRule)-[CONSENT]->(Identity')
 - GrantRule: (Identity)-[GRANT]->(GrantRule)-[GRANT]->(PublishRule), (GrantRule)-[GRANT]->(Identity), (GrantRule)-[ON_BEHALF_OF]->(Identity)

# Wins

## What does the system provide

The system will be able to provide all the basic functionalities of an Identity Provider, including some OAuth2 specific usecases:

  - Identity Authentication
  - Create an identity
  - Delete your identity
  - Recover identity when password gets lost
  - Invite others to join the system
  - Public or private sign up process
  - Change password
  - 2FA authentication using an Authenticator App
  - Create OAuth2 client issuing identifier and secret usable by OAuth2
  - Create Resource Server for publishing scopes and keeping track of system layout
  - Identity Logout

The Access and Authorization Provider is self contained and can in theory be used with any other IDP. A list of functionalities includes:

  - Manage and secure Identity consent to 3. party applications used by the OAuth2 authorization code flow
  - Manage access rights for identities to resource providers
  - Judge identities access to resource providers
  - Publish scopes for subscriptions (lets a resource provider define what a scope means)
  - Subscribe to published scopes (allow client x to use scope y for audience z)
  - Attribute Based Access Control merged with Role Based Access Control

## OAuth2
The system will support a number of the OAuth2 protocol flows:

 - Client Credentials: A trusted application that can keep a secret, meaning you control both the resource provider and the application access it
 - Authorization Code: A trusted application that requires user consent to access resource providers on behalf of the user
 - Authorization Code with PKCE: An untrusted application that is incapable of keeping a secret like mobile applications

These flows are all managed by ORY Hydra, but still need some set of support by the AAP.

## Trust and Control
The system will define a logical trust sphere surrounding the IDP, IDPUI, AAP and AAPUI services, that you decide how and where to run, and provide a security protocol that can be used by resource providers to secure access to their resources with developer defined levels of access control.

Hopefully this minimal system will make you rejoice when being able to take responsibility for this critical part of your infrastructure knowing that you are in complete control and nothing is left a black box or behind a prioritized paywall.
