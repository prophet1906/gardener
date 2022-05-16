---
title: "The OAuth 2.0 Authorization Framework : Conceptual Overview"
tags:
- oauth2
- framework
- security
- rfc
---

The OAuth 2.0 Authorization framework enables a third-party application to obtain limited access to an HTTP service, either on behalf of a resource owner by orchestrating an approval interaction between the resource owner and the HTTP service, or by allowing the third-party application to obtain access on its own behalf.

**Real world example** - Login/Signup with google button in third party websites like twitter, linkedin, etc use oauth 2.0 for authorization from your existing google account.

## Introduction
Following figure shows traditional client-server authentication model.

![[Excalidraw/oauth/traditional-client-server-auth.excalidraw.png]]

Problems & Limitations with this approach:
- Third-party applications store resource owner's credentials
- Resource servers are required to support password authentication
- Third-party applications gain overly broad access to resources because credentials are shared
- Resource owner's lack ability to restrict duration or access to subset of resources
- Resource owner's cannot revoke access to individual third party without changing credentials
- Compromise of third-party application results in compromise of the end-user's password and all data protected by that password

OAuth addresses these issues by introducing an authorization layer and separating the role of the client and resource owner. In OAuth, every client is issued different set of credentials than those of the resource owner. Instead of using the resource owner's credentials to access protected resources, the client obtains an access token -- a string denoting a specific scope, lifetime, and other access attributes. Access tokens are issued to third-party clients by an authorization server with the approval of the resource owner. The client uses the access token to access the protected resources hosted by the resource server.

### Roles
OAuth defines four roles:
1. **Resource owner** - An entity capable of granting access to a protected resource. When the resource owner is a person, it is referred to as an end-user. Example - Google account owner/end user
2. **Resource server** - The server hosting the protected resources, capable of accepting and responding to protected resource requests using access tokens. Example - [People API](https://developers.google.com/people/)
3. **Client** - An application making protected resource requests on behalf of the resource owner and with its authorization. The term "client" does not imply any particular implementation characteristics (e.g., wheather the application executes on a server, a desktop, or other devices). Example - Twitter web/android app
4. **Authorization Server** - The server issuing access tokens to the client after successfully authenticating the resource owner and obtaining authorization. Example - [Google Identity Service](https://developers.google.com/identity)

Real world example showing all roles - When you press login with google button in twitter(**Client**) login page, it redirects you to google(**Authorization Server**) login page. After logging into google, it requests your(**Resource Owner**) to allow access to your contacts from People API(**Resource Server**). Twitter uses this contact info to suggest people to follow.

### Protocol Flow
![[Excalidraw/oauth/abstract-protocol-flow.excalidraw.png]]

The figure above illustrates abstract OAuth 2.0 flow with interaction between the 4 roles and includes following steps:

<ol type="A">
	<li>Client request authorization from request owner</li>
	<li>Client receives authorization grant, which is a credential representing the resource owner's authorization.</li>
	<li>Client requests an access token by authenticating with authorization server and presenting authorization grant</li>
	<li>Authorization server authenticates client and validates authorization grant, and if valid, issues an access token.</li>
	<li>Client requests protected resource from resource server presenting access token</li>
	<li>Resource server validates access token, and if valid, server the request</li>
</ol>

### Authorization Grant
Authorization grant is a credential representing resource owner's authorization used by the client to obtain an access token.

There are 4 types of authorization grant:
1. [[#Authorization Code Grant]]
2. [[#Implicit Grant]]
3. [[#Resource Owner Password Credentials Grant]]
4. [[#Client Credentials Grant]]

[RFC 6749: The OAuth 2.0 Authorization Framework)](https://www.rfc-editor.org/rfc/rfc6749.html) also provides extensibility mechanism for defining additional types.

### Access Token
Access tokens are credentials used to access protected resources. An access token is a string representing an authorization issues to the client. The string is usually opaque to the client. Tokens represent specific scopes and durations of access, granted by resource owner, and enforced by resource server and authorization server.

### Refresh Token
Refresh tokens are credentials used to obtain access tokens. Refresh tokens are issued to the client by the auhorization server and are used to obtain a new access token when current access token becomes invalid ot expires, or to obtain additional access token with narrower scope (access tokens may have shorter lifetime and fewer permissions than authorized by resource owner).

<span style="color: red;">
Refresh tokens are intended for use only with authorization server and are never sent to resource servers.
</span>

Following figure illustrates how refresh tokens are used to get new access token.

![[Excalidraw/oauth/refresh-expired-access-token.excalidraw.png]]

## Client Registration
Before initiating the protocol, client registers itself with authorization server. This process is beyond the scope of this note but typically involves end-user interaction with an HTML registration form. 

Example - Google Login Page after getting redirected by a website

When registering a client, the client developer shall:
- specify the client type
- provide its client redirection URIs
- include any other information required by the authorization server (e.g., application name, website, description, logo image, the acceptance of legal terms)

### Client Types
OAuth defines 2 client types, based on their ability to authenticate securely with the authorization server (i.e., ability to maintain the confidentiality of their client credentials):

- **confidential** : Clients capable of maintaining the confidentiality of their credentials (e.g., client implemented on a secure server with restricted access to client credentials), or capable of secure client authentication using other means. 
- **public** : Client incapable of maintaining the confidentiality if their credentials (e.g., clients executing on the device used by the resource owner, such as an installed native application or a web browser-based application), and incapable of secure client authentication via any other means.

<span style="color: red;">
Authorization server SHOULD NOT make assumptions about the client type.
</span>

Client type specification mentioned above was designed around the following client profiles:
- **Web application** : A web application is a confidential client running on a web server. Resource owners access the client via an HTML user interface rendered in a user-agent on the device used by the resource owner. The client credentials as well as any access token issued to client are stored on the web server and are not exposed to or accessible by resource owner.
- **User-agent-based application** : A user-agent-based application is a public client in which the client code is downloaded from a web server and executes within a user-agent(e.g., web browser) on the device used by the resource owner. Protocol data and credentials are easily accessible (and often visible) to resource owner. Since such applications reside within the user-agent, they can make seamless use of the user-agent capabilities when requesting authorization.
- **Native application** : A native application is a public client installed and executed on the device used by the resource owner. Protocol data and credentials are accessible to the resource owner. It is assumed that any client authentication credentials included in the application can be extracted. On the other hand, dynamically issued credentials such as access tokens or refresh tokens can receive acceptable level of protection. At a minimum, these credentials are protected from hostile servers with which the application may interact. On some platforms, these credentials might be protected from other applications residing on the same device.

### Client Identifier
Authorization server issues a unique client identifier to every registered client. Client identifier is not a secret and is exposed to resource owner.

### Client Authentication
For confidential client type, client and authorization server establish client authentication method suitable for the security requirement of the authorization server.

Confidential clients are typically issues a set of client credentials used for authenticating with the authorization server (e.g., password, public/private key pair). **Example** - Applications in Cloud Foundry may use client credentials present in **VCAP_SERVICES** for any oauth secured service.

## Protocol Endpoints
The authorization process utilizes 2 authorization server endpoints (HTTP resources):
- **Authorization Endpoint** - used by the client to obtain authorization from the resource owner via user-agent redirection.
- **Token Endpoint** - used by the client to exchange an authorization grant for an access token, typically with client authentication.

<span style="color: red;">
Not every authorization grant type utilizes both endpoints.
</span>

As well as one client endpoint:
- **Redirection endpoint** - used by the authorization server to return responses containing authorization credentials to the client via the resource owner user-agent.

## Obtaining Authorization
To request an access token, the client obtains authorization from the resource owner. The authorization is expressed in the form of an authorization grant, which client uses to request access token.

### Authorization Code Grant

The authorization code grant type is used to obtain both access tokens and refresh tokens and is optimized for confidential clients.
Since this is a redirection-based flow, the client must be capable of interacting with the resource owner's user-agent (typically a web browser) and capable of receiving incoming requests (via redirection) from the authorization server. 

The Authorization Code grant can be used anytime you connect a user profile to one or multiple accounts from within a website and for instance you authorize the website to access information owned from LinkedIn, Facebook, Twitter, etc….even all at the same time.

Example - Connecting linkedin account to twitter account and allowing linkedin to access your twitter timeline information.

![[Excalidraw/oauth/authorization-code-flow.excalidraw.png]]
<span style="color: red;">
The lines illustrating steps A, B, C are broken into two parts as they pass through the user-agent.
</span>


### Implicit Grant

The implicit grant type is used to obtain access tokens(without any support for refresh token) and is optimized for public clients known to operate a particular redirection URI. These clients are typically implemented in a browser using a scripting language such as Javascript.

Since this is a redirection-based flow, the client must be capable of interacting with the resource owner's user-agent (typically a web browser) and capable of receiving incoming requests (via redirection) from the authorization server.

Unlike the authorization code grant type, in which the client makes separate requests for authorization and for an access token, the client receives the access token as the result of the authorization request.

The implicit grant type does not include client authentication, and relies on the presence of the resource owner and the registration of the redirection URI.  

The Implicit Grant is very similar to the Authorization grant , but is not as secure as the first one. Another application running on the same machine / device could steal the authorization information exchanged between the client and the third party library and use it.

A malicious client can impersonate another client and obtain access to protected resources, if the impersonated client fails to, or is unable to, keep its client credentials confidential. For more details read [draft-ietf-oauth-v2-31](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-v2-31#section-10.2).


Example - Applications leaving context and opening new window to authenticate. Real examples include android web apps & some websites redirecting you to another website in new tab. The control moves from your application to another website in  new tab, thus making it very risky.

<span style="color: red;">
The access token is encoded into the redirection URI, it may be exposed to the resource owner and other applications residing on the same device.
</span>

![[Excalidraw/oauth/implicit-grant-flow.excalidraw.png]]
<span style="color: red;">
The lines illustrating steps A and B are broken into two parts as they pass through the user-agent.
</span>

### Resource Owner Password Credentials Grant

The resource owner password credentials grant type is suitable in cases where the resource owner has a trust relationship with the client, such as the device operating system or a highly privileged application.

This grant type is suitable for clients capable of obtaining the resource owner's credentials (username and password, typically using an interactive form).  It is also used to migrate existing clients using direct authentication schemes such as HTTP Basic or Digest authentication to OAuth by converting the stored credentials to an access token.

Example - Standalone applications in your device like netflix app in andorid phone.

<span style="color: red;">
 The authorization server should take special care when enabling this grant type and only allow it when other flows are not viable.
</span>

![[Excalidraw/oauth/resource-owner-password-credentials-flow.excalidraw.png]]

### Client Credentials Grant

 The client can request an access token using only its client credentials (or other supported means of authentication) when the client is requesting access to the protected resources under its control, or those of another resource owner that have been previously arranged with the authorization server).

Example - Application Hosted in Cloud Foundry authenticating to get access to backing service.

<span style="color: red;">
This flow doesn't involve any end-user and should only be used for machine-to-machine authentication in order to allow some server side code to access some protected resource like third-party Web API.
</span>

![[Excalidraw/oauth/client-credentials-flow.excalidraw.png]]

## Conclusion
I didn't cover authorization request/response, access token request/response formats and security considerations here. Please refer RFC for getting detailed request/response formats. The OAuth 2.0 Core Framework (RFC 6749) only defines roles and a base level of functionality. Implementation details are left unspecified as out of scope for RFC. OAuth Working Group has published additional RFCs built on top of core framework to fill missing gaps. I would try covering some of those RFCs in future notes. 

This note only covers essential content from [RFC 6749: The OAuth 2.0 Authorization Framework](https://www.rfc-editor.org/rfc/rfc6749.html).
