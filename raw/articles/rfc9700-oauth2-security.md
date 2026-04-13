<!-- source_url: https://oauth.net/2/oauth-best-practice/ -->
RFC 9700: Best Current Practice for OAuth 2.0 Security
RFC 9700
OAuth 2.0 Security BCP
January 2025
Lodderstedt, et al.
Best Current Practice
[Page]
Stream:
Internet Engineering Task Force (IETF)
RFC:
9700
BCP:
240
Updates:
6749
,
6750
,
6819
Category:
Best Current Practice
Published:
January 2025
ISSN:
2070-1721
Authors:
T. Lodderstedt
SPRIND
J. Bradley
Yubico
A. Labunets
Independent Researcher
D. Fett
Authlete
RFC 9700
Best Current Practice for OAuth 2.0 Security
Abstract
This document describes best current security practice for OAuth 2.0. It updates
and extends the threat model and security advice given in RFCs 6749, 6750, and 6819 to incorporate practical experiences gathered since
OAuth 2.0 was published and covers new threats relevant due to the broader
application of OAuth 2.0. Further, it deprecates some modes of operation that are
deemed less secure or even insecure.
¶
Status of This Memo
This memo documents an Internet Best Current Practice.
¶
This document is a product of the Internet Engineering Task Force
            (IETF).  It represents the consensus of the IETF community.  It has
            received public review and has been approved for publication by
            the Internet Engineering Steering Group (IESG).  Further information
            on BCPs is available in Section 2 of RFC 7841.
¶
Information about the current status of this document, any
            errata, and how to provide feedback on it may be obtained at
https://www.rfc-editor.org/info/rfc9700
.
¶
Copyright Notice
Copyright (c) 2025 IETF Trust and the persons identified as the
            document authors. All rights reserved.
¶
This document is subject to BCP 78 and the IETF Trust's Legal
            Provisions Relating to IETF Documents
            (
https://trustee.ietf.org/license-info
) in effect on the date of
            publication of this document. Please review these documents
            carefully, as they describe your rights and restrictions with
            respect to this document. Code Components extracted from this
            document must include Revised BSD License text as described in
            Section 4.e of the Trust Legal Provisions and are provided without
            warranty as described in the Revised BSD License.
¶
▲
Table of Contents
1.
Introduction
Since its publication in
[
RFC6749
]
and
[
RFC6750
]
, OAuth 2.0 (referred to as simply "OAuth" in this document) has gained massive traction in the market
and became the standard for API protection and the basis for federated
login using OpenID Connect
[
OpenID.Core
]
. While OAuth is used in a
variety of scenarios and different kinds of deployments, the following
challenges can be observed:
¶
OAuth implementations are being attacked through known implementation
  weaknesses and anti-patterns (i.e., well-known patterns that are considered
insecure). Although most of these threats are discussed in the OAuth 2.0
  Threat Model and Security Considerations
[
RFC6819
]
, continued exploitation
 demonstrates a need for more specific recommendations, easier to implement
  mitigations, and more defense in depth.
¶
OAuth is being used in environments with higher security requirements than
considered initially, such as open banking, eHealth, eGovernment, and
electronic signatures. Those use cases call for stricter guidelines and
additional protection.
¶
OAuth is being used in much more dynamic setups than originally anticipated,
  creating new challenges with respect to security. Those challenges go beyond
  the original scope of
[
RFC6749
]
,
[
RFC6750
]
, and
[
RFC6819
]
.
¶
OAuth initially assumed static relationships between clients,
authorization servers, and resource servers. The URLs of the servers were
known to the client at deployment time and built an anchor for the
trust relationships among those parties. The validation of whether the
client is talking to a legitimate server was based on TLS server
authentication (see
Section 4.5.4
of [
RFC6819
]
). With the increasing
adoption of OAuth, this simple model dissolved and, in several
scenarios, was replaced by a dynamic establishment of the relationship
between clients on one side and the authorization and resource servers
of a particular deployment on the other side. This way, the same
client could be used to access services of different providers (in
case of standard APIs, such as email or OpenID Connect) or serve as a
front end to a particular tenant in a multi-tenant environment.
Extensions of OAuth, such as the OAuth 2.0 Dynamic Client Registration
Protocol
[
RFC7591
]
and OAuth 2.0 Authorization Server Metadata
[
RFC8414
]
were developed to support the use of OAuth in
dynamic scenarios.
¶
Technology has changed. For example, the way browsers treat fragments when
  redirecting requests has changed, and with it, the implicit grant's
  underlying security model.
¶
This document provides updated security recommendations to address these
challenges. It introduces new requirements beyond those defined in existing
specifications such as OAuth 2.0
[
RFC6749
]
and OpenID Connect
[
OpenID.Core
]
and deprecates some modes of operation that are deemed less secure or even
insecure. However, this document does not supplant the security advice given in
[
RFC6749
]
,
[
RFC6750
]
, and
[
RFC6819
]
, but complements those documents.
¶
Naturally, not all existing ecosystems and implementations are
compatible with the new requirements, and following the best practices described in
this document may break interoperability. Nonetheless, it is
RECOMMENDED
that
implementers upgrade their implementations and ecosystems as soon as feasible.
¶
OAuth 2.1, under development as
[
OAUTH-V2.1
]
, will incorporate
security recommendations from this document.
¶
1.1.
Structure
The remainder of this document is organized as follows:
Section 2
summarizes the most important best practices for every OAuth implementer.
Section 3
presents the updated OAuth attacker model.
Section 4
is a
detailed analysis of the threats and implementation issues that can be found in
the wild (at the time of writing) along with a discussion of potential countermeasures.
¶
1.2.
Conventions and Terminology
The key words "
MUST
", "
MUST NOT
",
    "
REQUIRED
", "
SHALL
", "
SHALL NOT
",
    "
SHOULD
", "
SHOULD NOT
",
    "
RECOMMENDED
", "
NOT RECOMMENDED
",
    "
MAY
", and "
OPTIONAL
" in this document are to be
    interpreted as described in BCP 14
[
RFC2119
]
[
RFC8174
]
when, and only when, they appear in all capitals, as
    shown here.
¶
This specification uses the terms "access token", "authorization
endpoint", "authorization grant", "authorization server", "client",
"client identifier" (client ID), "protected resource", "refresh
token", "resource owner", "resource server", and "token endpoint"
defined by OAuth 2.0
[
RFC6749
]
.
¶
An "open redirector" is an endpoint on a web server that forwards a user's
browser to an arbitrary URI obtained from a query parameter.
¶
2.
Best Practices
This section describes the core set of security mechanisms and measures that
are considered to be best practices at the time of writing. Details
about these security mechanisms and measures (including detailed attack
descriptions) and requirements for less commonly used options are provided in
Section 4
.
¶
2.1.
Protecting Redirect-Based Flows
When comparing client redirection URIs against pre-registered URIs, authorization
servers
MUST
utilize exact string matching except for port numbers in
localhost
redirection URIs of native apps (see
Section 4.1.3
). This
measure contributes to the prevention of leakage of authorization codes and
access tokens (see
Section 4.1
). It can also help to detect
mix-up attacks (see
Section 4.4
).
¶
Clients and authorization servers
MUST NOT
expose URLs that forward the user's browser to
arbitrary URIs obtained from a query parameter (open redirectors) as
described in
Section 4.11
. Open redirectors can enable
exfiltration of authorization codes and access tokens.
¶
Clients
MUST
prevent Cross-Site Request Forgery (CSRF). In this
context, CSRF refers to requests to the redirection endpoint that do
not originate at the authorization server, but at a malicious third party
(see
Section 4.4.1.8
of [
RFC6819
]
for details). Clients that have
ensured that the authorization server supports Proof Key for Code Exchange (PKCE)
[
RFC7636
]
MAY
rely on the CSRF protection provided by PKCE. In OpenID Connect flows,
the
nonce
parameter provides CSRF protection. Otherwise, one-time
use CSRF tokens carried in the
state
parameter that are securely
bound to the user agent
MUST
be used for CSRF protection (see
Section 4.7.1
).
¶
When an OAuth client can interact with more than one authorization server, a
defense against mix-up attacks (see
Section 4.4
) is
REQUIRED
. To this end, clients
SHOULD
¶
use the
iss
parameter as a countermeasure according to
[
RFC9207
]
, or
¶
use an alternative countermeasure based on an
iss
value in the
authorization response (such as the
iss
claim in the ID Token in
[
OpenID.Core
]
or in
[
OpenID.JARM
]
responses), processing that value as described in
[
RFC9207
]
.
¶
In the absence of these options, clients
MAY
instead use distinct redirection URIs
to identify authorization endpoints and token endpoints, as described in
Section 4.4.2
.
¶
An authorization server that redirects a request potentially containing user credentials
MUST
avoid forwarding these user credentials accidentally (see
Section 4.12
for details).
¶
2.1.1.
Authorization Code Grant
Clients
MUST
prevent authorization code
injection attacks (see
Section 4.5
) and misuse of authorization codes using one of the following options:
¶
Public clients
MUST
use PKCE
[
RFC7636
]
to this end, as motivated in
Section 4.5.3.1
.
¶
For confidential clients, the use of PKCE
[
RFC7636
]
is
RECOMMENDED
, as it
provides strong protection against misuse and injection of authorization
codes as described in
Section 4.5.3.1
. Also, as a side effect,
it prevents CSRF even in the presence of strong attackers as described in
Section 4.7.1
.
¶
With additional precautions, described in
Section 4.5.3.2
,
confidential OpenID Connect
[
OpenID.Core
]
clients
MAY
use the
nonce
parameter and the
respective Claim in the ID Token instead.
¶
In any case, the PKCE challenge or OpenID Connect
nonce
MUST
be
transaction-specific and securely bound to the client and the user agent in
which the transaction was started.
Authorization servers are encouraged to make a reasonable effort at detecting and
preventing the use of constant values for the PKCE challenge or OpenID Connect
nonce
.
¶
Note: Although PKCE was designed as a mechanism to protect native
apps, this advice applies to all kinds of OAuth clients, including web
applications.
¶
When using PKCE, clients
SHOULD
use PKCE code challenge methods that
do not expose the PKCE verifier in the authorization request.
Otherwise, attackers that can read the authorization request (cf. Attacker
(A4)
in
Section 3
) can break the security provided
by PKCE. Currently,
S256
is the only such method.
¶
Authorization servers
MUST
support PKCE
[
RFC7636
]
.
¶
If a client sends a valid PKCE
code_challenge
parameter in the
authorization request, the authorization server
MUST
enforce the correct usage
of
code_verifier
at the token endpoint.
¶
Authorization servers
MUST
mitigate PKCE downgrade attacks by ensuring that a
token request containing a
code_verifier
parameter is accepted only if a
code_challenge
parameter was present in the authorization request; see
Section 4.8.2
for details.
¶
Authorization servers
MUST
provide a way to detect their support for
PKCE. It is
RECOMMENDED
for authorization servers to publish the element
code_challenge_methods_supported
in their Authorization Server Metadata
[
RFC8414
]
containing the supported PKCE challenge methods (which can be used by
the client to detect PKCE support). Authorization servers
MAY
instead provide a
deployment-specific way to ensure or determine PKCE support by the authorization server.
¶
2.1.2.
Implicit Grant
The implicit grant (response type
token
) and other response types
causing the authorization server to issue access tokens in the
authorization response are vulnerable to access token leakage and
access token replay as described in Sections
4.1
,
4.2
,
4.3
, and
4.6
.
¶
Moreover, no standardized method for sender-constraining exists to
bind access tokens to a specific client (as recommended in
Section 2.2
) when the access tokens are issued in the
authorization response. This means that an attacker can use the leaked or stolen
access token at a resource endpoint.
¶
In order to avoid these issues, clients
SHOULD NOT
use the implicit
grant (response type
token
) or other response types issuing
access tokens in the authorization response, unless access token injection
in the authorization response is prevented and the aforementioned token leakage
vectors are mitigated.
¶
Clients
SHOULD
instead use the response type
code
(i.e., authorization
code grant type) as specified in
Section 2.1.1
or any other response type that
causes the authorization server to issue access tokens in the token
response, such as the
code id_token
response type. This allows the
authorization server to detect replay attempts by attackers and
generally reduces the attack surface since access tokens are not
exposed in URLs. It also allows the authorization server to
sender-constrain the issued tokens (see
Section 2.2
).
¶
2.2.
Token Replay Prevention
2.2.1.
Access Tokens
A sender-constrained access token scopes the applicability of an access
token to a certain sender. This sender is obliged to demonstrate knowledge
of a certain secret as a prerequisite for the acceptance of that token at
the recipient (e.g., a resource server).
¶
Authorization and resource servers
SHOULD
use mechanisms for sender-constraining
access tokens, such as mutual TLS for OAuth 2.0
[
RFC8705
]
or OAuth 2.0
Demonstrating Proof of Possession (DPoP)
[
RFC9449
]
(see
Section 4.10.1
), to prevent misuse of stolen and leaked access tokens.
¶
2.2.2.
Refresh Tokens
Refresh tokens for public clients
MUST
be sender-constrained or use refresh
token rotation as described in
Section 4.14
.
[
RFC6749
]
already
mandates that refresh tokens for confidential clients can only be used by the
client for which they were issued.
¶
2.3.
Access Token Privilege Restriction
The privileges associated with an access token
SHOULD
be restricted to
the minimum required for the particular application or use case. This
prevents clients from exceeding the privileges authorized by the
resource owner. It also prevents users from exceeding their privileges
authorized by the respective security policy. Privilege restrictions
also help to reduce the impact of access token leakage.
¶
In particular, access tokens
SHOULD
be audience-restricted to a specific resource
server or, if that is not feasible, to a small set of resource servers. To put this into effect, the authorization server associates
the access token with certain resource servers, and every resource
server is obliged to verify, for every request, whether the access
token sent with that request was meant to be used for that particular
resource server. If it was not, the resource server
MUST
refuse to serve the
respective request. The
aud
claim as defined in
[
RFC9068
]
MAY
be
used to audience-restrict access tokens. Clients and authorization servers
MAY
utilize the
parameters
scope
or
resource
as specified in
[
RFC6749
]
and
[
RFC8707
]
, respectively, to determine the
resource server they want to access.
¶
Additionally, access tokens
SHOULD
be restricted to certain resources
and actions on resource servers or resources. To put this into effect,
the authorization server associates the access token with the
respective resource and actions and every resource server is obliged
to verify, for every request, whether the access token sent with that
request was meant to be used for that particular action on the
particular resource. If not, the resource server must refuse to serve
the respective request. Clients and authorization servers
MAY
utilize
the parameter
scope
as specified in
[
RFC6749
]
and
authorization_details
as specified in
[
RFC9396
]
to determine those
resources and/or actions.
¶
2.4.
Resource Owner Password Credentials Grant
The resource owner password credentials grant
[
RFC6749
]
MUST NOT
be used. This grant type insecurely exposes the credentials of the resource
owner to the client. Even if the client is benign, usage of this grant results in an increased
attack surface (i.e., credentials can leak in more places than just the authorization server) and in training users to enter their credentials in places other than the authorization server.
¶
Furthermore, the resource owner password credentials grant is not designed to
work with two-factor authentication and authentication processes that require
multiple user interaction steps. Authentication with cryptographic credentials
(cf. WebCrypto
[
W3C.WebCrypto
]
, WebAuthn
[
W3C.WebAuthn
]
) may be impossible
to implement with this grant type, as it is usually bound to a specific web origin.
¶
2.5.
Client Authentication
Authorization servers
SHOULD
enforce client authentication if it is feasible, in
the particular deployment, to establish a process for issuance/registration of
credentials for clients and ensuring the confidentiality of those credentials.
¶
It is
RECOMMENDED
to use asymmetric cryptography for
client authentication, such as mutual TLS for OAuth 2.0
[
RFC8705
]
or signed JWTs
("Private Key JWT") in accordance with
[
RFC7521
]
and
[
RFC7523
]
. The latter is defined in
[
OpenID.Core
]
as the client authentication method
private_key_jwt
).
When asymmetric cryptography for client authentication is used, authorization
servers do not need to store sensitive symmetric keys, making these
methods more robust against leakage of keys.
¶
2.6.
Other Recommendations
The use of OAuth Authorization Server Metadata
[
RFC8414
]
can help to improve the security of OAuth
deployments:
¶
It ensures that security features and other new OAuth features can be enabled
automatically by compliant software libraries.
¶
It reduces chances for misconfigurations -- for example, misconfigured endpoint
URLs (that might belong to an attacker) or misconfigured security features.
¶
It can help to facilitate rotation of cryptographic keys and to ensure
cryptographic agility.
¶
It is therefore
RECOMMENDED
that authorization servers publish OAuth Authorization Server Metadata according to
[
RFC8414
]
and that clients make use of this Authorization Server Metadata (when available) to configure themselves.
¶
Under the conditions described in
Section 4.15.1
,
authorization servers
SHOULD NOT
allow clients to influence their
client_id
or
any other claim that could cause confusion with a genuine resource owner.
¶
It is
RECOMMENDED
to use end-to-end TLS according to
[
BCP195
]
between the client and the resource server. If TLS
traffic needs to be terminated at an intermediary, refer to
Section 4.13
for further security advice.
¶
Authorization responses
MUST NOT
be transmitted over unencrypted network
connections. To this end, authorization servers
MUST NOT
allow redirection URIs that use the
http
scheme except for native clients that use loopback interface redirection as
described in
Section 7.3
of [
RFC8252
]
.
¶
If the authorization response is sent with in-browser communication techniques
like postMessage
[
WHATWG.postmessage_api
]
instead of HTTP redirects, both the
initiator and receiver of the in-browser message
MUST
be strictly verified as described
in
Section 4.17
.
¶
To support browser-based clients, endpoints directly accessed by such clients
including the Token Endpoint, Authorization Server Metadata Endpoint,
jwks_uri
Endpoint, and Dynamic Client Registration Endpoint
MAY
support the use of
Cross-Origin Resource Sharing (CORS)
[
WHATWG.CORS
]
.
However, CORS
MUST NOT
be
supported at the authorization endpoi

[... truncated, 112958 chars total]