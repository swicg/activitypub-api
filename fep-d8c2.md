---
slug: "d8c2"
authors: Evan Prodromou <evanp@socialwebfoundation.org>
status: DRAFT
dateReceived: 2023-09-17
trackingIssue: https://codeberg.org/fediverse/fep/issues/165
discussionsTo: https://codeberg.org/evanp/fep/issues
---
# FEP-d8c2: OAuth 2.0 Profile for the ActivityPub API

## Summary

This FEP defines a mechanism for using an ActivityPub object ID as the `client_id` in the OAuth 2.0 authorization code flow.

(An earlier version defined a full profile for using OAuth 2.0 with the
ActivityPub API, but this version has been abbreviated to focus only on the
client ID mechanism. The title has been retained to accommodate FEP tooling.)

## Motivation

[ActivityPub] defines the ActivityPub API, a RESTful HTTP API for stream-oriented social software. This API allows client software to read ActivityPub objects, including actors, collections, activities, and content objects. Client software can also create new `Activity` objects by posting to an actor's `outbox` collection (also called "client-to-server" or "c2s").

The ActivityPub specification does not define an authorization mechanism for the API, although the [ActivityPub Primer Authorization and Authentication][ActivityPubAuth] recommendations include some suggestions. Although there are many ways to implement client authorization for an API, OAuth 2.0 is a popular and well-understood framework.

OAuth 2.0 is broad and encompasses a number of different techniques and use cases. [OAuth 2.0 Simplified][OAuth20Simplified] documents the most common profile of OAuth 2.0: [authorization code flow](https://datatracker.ietf.org/doc/html/rfc6749#section-4.1) and bearer tokens. Many OAuth 2.0 client libraries implement this profile.

The OAuth 2.0 authorization code flow requires two main endpoints for a client to initiate the flow: an authorization endpoint and a token endpoint. These can be discovered using the `endpoints` property of the ActivityPub actor or the [Authorization Server Metadata][AuthorizationServerMetadata] endpoint from RFC 8414.

The OAuth 2.0 flow uses a *client identifier* to show important information about the client software to the user, and to avoid certain classes of spoofing attacks.

A common use case for OAuth 2.0 is an API supplied by a single provider. With a single provider, the client developer can register a client ID out of band using the provider's developer Web site or other tools.

With multiple providers, as with the Fediverse, out-of-band registration becomes untenable. With tens of thousands of known ActivityPub servers on the Internet, client developers cannot manually register client IDs with each provider of the ActivityPub API.

One option is to use [Dynamic Client Registration][DynamicClientRegistration] protocol from RFC 7591. This defines a standard HTTP endpoint used for registering an application with an authorization server and receiving a unique client identifier.

Dynamic client registration adds some extra complexity on the client side. In particular, client software has to maintain a record of the correct client ID for each authorization server used.

This profile addresses these issues by using a single, well-defined ActivityPub object to identify and describe the client software.

## Client identifier

ActivityPub provides a rich vocabulary for describing objects in the social space. Each object in the ActivityPub world has a unique https: URI, which must be dereferenceable to a JSON-LD document describing the object.

This allows a distributed description of ActivityPub API clients that doesn't require out-of-band registration.

Objects dereferenced at the id SHOULD be of type `Application` or `Service`. They MUST have an `id` property with the same value as the `client_id` parameter. They MUST have a `redirectURI` property with the redirect URI for the client (see [Context document](#context-document) below).

Clients SHOULD provide metadata to help users make authorization decisions, including:

- `nameMap` or `name`: The name of the client software.
- `icon`: An `Image` object with the icon for the client software.
- `summaryMap` or `summary`: A description of the application or service.
- `attributedTo`: The `name`, `id`, `icon` and `summary` properties of
  the actor responsible for the client software.

## Discovery

Support for using ActivityPub object IDs as OAuth 2.0 client IDs can be declared in two ways.

### Actor discovery

An ActivityPub actor can include the `objectIDAsClientID` property. If `true`, client software can use the client ID format in this specification to identify themselves to authorization servers.

### Authorization Server Metadata

An authorization server can declare its support for ActivityPub object IDs as client IDs by adding the `activitypub_object_id_as_client_id` flag to its [Authorization Server Metadata][AuthorizationServerMetadata].

## Context document

The context document for this specification is at `https://purl.archive.org/socialweb/oauth/2.0`. Its contents are as follows:

```json
{
  "@context": {
    "oauth": "https://purl.archive.org/socialweb/oauth#",
    "redirectURI": {
      "@id": "oauth:redirectURI",
      "@type": "xsd:anyURI"
    },
    "objectIDAsClientID": {
      "@id": "oauth:objectIDAsClientID",
      "@type": "xsd:boolean"
    }
  }
}
```

### Context URL aliases

Aliases are provided for the context URL to allow change over time with backwards compatibility, using a [semantic versioning](https://semver.org/) strategy.

- `https://purl.archive.org/socialweb/oauth/2.0.0` This URL will be bytewise stable, and can be used for clients that use digital signatures or hashes to validate context URLs.
- `https://purl.archive.org/socialweb/oauth/2.0` The preferred URL. Backwards-compatible changes, such as whitespace and formatting, may be made, but no new terms will be added and none will be removed or modified. Will be kept up to date with the latest 2.0.x version.
- `https://purl.archive.org/socialweb/oauth/2` New terms may be added, but none will be removed or modified. Will be kept up to date with the latest 2.x.x version.
- `https://purl.archive.org/socialweb/oauth` The latest version of the context document; backwards-incompatible changes may be applied, such as removing or modifying terms.

New versions of the context document will increment the major, minor and patch version as needed.

## Properties

### redirectURI

The `redirectURI` property is an IRI that the client uses to receive the authorization code after the user authorizes the client. The server MUST verify that the `redirect_uri` parameter in the authorization request matches the `redirectURI` property of the client object.

### objectIDAsClientID

This flag has a boolean value, `true` or `false`. If true, the authorization server for the actor with this property supports using ActivityPub object IDs
as client IDs, as described in this document.

## Examples

### Actor flag

The following actor description declares that the actor's authorization server supports ActivityPub object IDs as OAuth 2.0 client IDs using the `objectIDAsClientID` flag.

```json
{
  "@context": [
    "https://www.w3.org/ns/activitystreams",
    "https://purl.archive.org/socialweb/oauth/2.0"
  ],
  "id": "https://social.example/user/evan",
  "inbox": "https://social.example/user/evan/inbox",
  "outbox": "https://social.example/user/evan/outbox",
  "endpoints": {
    "oauthAuthorizationEndpoint": "https://social.example/authorize",
    "oauthTokenEndpoint": "https://social.example/token"
  },
  "objectIDAsClientID": true
}
```

### Authorization Server Metadata flag

An authorization server can declare its support for using ActivityPub object IDs as client IDs with the `activitypub_object_id_as_client_id` flag.

```json
{
  "issuer": "https://social.example",
  "authorization_endpoint": "https://social.example/authorize",
  "token_endpoint": "https://social.example/token",
  "registration_endpoint": "https://social.example/registration",
  "scopes_supported": [
    "read",
    "write"
  ],
  "response_types_supported": [
    "code"
  ],
  "grant_types_supported": [
    "authorization_code",
    "refresh_token"
  ],
  "code_challenge_methods_supported": [
    "S256"
  ],
  "token_endpoint_auth_methods_supported": [
    "none"
  ],
  "activitypub_object_id_as_client_id": true
}
```

### Follower recommender

A Web service that wants to use the ActivityPub API would define an ActivityPub object at `https://followrec.example/client`. This object has a `redirectURI` property with the URI of the Web application's authorization endpoint.

```json
{
  "@context": [
    "https://www.w3.org/ns/activitystreams",
    "https://purl.archive.org/socialweb/oauth/2.0"
  ],
  "id": "https:/followrec.example/apps/myapp",
  "name": "Follow Recommender",
  "type": "Service",
  "icon": {
    "type": "Image",
    "url": "http://followrec.example/followrec.png",
    "width": 256,
    "height": 256
  },
  "summaryMap": {
    "en": "Follow Recommender is a service that recommends people to follow based on your existing community."
  },
  "attributedTo": {
    "name": "Alyssa P. Hacker",
    "id": "https://hackers.example/alyssa",
    "type": "Person",
    "icon": {
      "type": "Image",
      "url": "https://hackers.example/alyssa/icon.png",
      "width": 256,
      "height": 256
    },
    "summaryMap": {
      "en": "Alyssa P. Hacker builds cool stuff on the Internet."
    }
  },
  "redirectURI": "https://followrec.example/oauth/callback"
}
```

### Mobile checkin app

An iOS app uses the ActivityPub API to post location updates for a user. Because the app is a native program, it uses a static site provided by its version control system to host the client object at `https://developer.git.example/kfc/client.json`.

```json
{
  "@context": [
    "https://www.w3.org/ns/activitystreams",
    "https://purl.archive.org/socialweb/oauth/2.0"
  ],
  "id": "https://developer.git.example/kfc/client.json",
  "name": "Kentucky Fried Checkin",
  "type": "Application",
  "icon": {
    "type": "Image",
    "url": "https://developer.git.example/kfc/icon.png",
    "width": 256,
    "height": 256
  },
  "summaryMap": {
    "en": "Kentucky Fried Checkin is a mobile app that allows you to post checkins to your ActivityPub timeline."
  },
  "attributedTo": {
    "name": "MobileCorp",
    "id": "https://mobilecorp.example/organization",
    "type": "Organization",
    "icon": {
      "type": "Image",
      "url": "https://mobilecorp.example/organization/logo.png",
      "width": 256,
      "height": 256
    },
    "summaryMap": {
      "en": "MobileCorp provides cool apps supporting the social web."
    }
  },
  "redirectURI": "checkin:oauth/callback"
}
```

Note that the `redirectURI` property is a custom URI scheme for the mobile app.

## Security considerations

- [OAuth 2.0 Security Best Current Practice](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-security-topics) provides a number of best practices for implementing OAuth 2.0.
- One risk of implementing OAuth 2.0 is that the user is redirected to the `redirect_uri` parameter after authorization is complete. This can be used as an attack to treat the authorization server as an open redirector. An app using an OAuth 2.0 authorization endpoint as an open redirector could change the client description document to have a different `redirectURI` for each request. One mitigation would be to archive the `redirectURI` value for each client, and cancel the flow if the value has changed too often.
- As with any protocol that requires fetching a client-provided URI, the server should take care in dereferencing the `client_id` parameter to avoid attacks such as very large responses, responses that take a long time to generate, or responses with poorly-formatted content.
- The ActivityPub object used to define the client includes metadata that can be spoofed, like the `name` or `icon`. An attacker could use the name, icon, or publisher of a popular application to trick users into authorizing the attacker's application. Tools such as shared blocklists, reputation systems, and user education can mitigate this risk.

## IANA Considerations

### OAuth Authorization Server Metadata Registry

The following authorization server metadata value is defined by this
specification and registered in the IANA "OAuth Authorization Server
Metadata" registry established in OAuth 2.0 Authorization Server
Metadata [RFC8414][AuthorizationServerMetadata].

- Metadata Name: activitypub_object_id_as_client_id
- Metadata Description: Boolean value specifying whether the authorization server supports using ActivityPub object IDs as client IDs.
- Change Controller: W3C Social Web Incubator Community Group
- Specification Document: https://fediverse.codeberg.page/fep/fep/d8c2/

## References

- Dick Hardt, [The OAuth 2.0 Authorization Framework][OAuth2], 2012
- Christine Lemmer Webber, Jessica Tallon, [ActivityPub][ActivityPub], 2018

[OAuth2]: https://www.ietf.org/rfc/rfc6749.txt
[AuthorizationServerMetadata]: https://www.ietf.org/rfc/rfc8414.txt
[DynamicClientRegistration]: https://datatracker.ietf.org/doc/html/rfc7591
[OAuth20Simplified]: https://www.oauth.com/
[ActivityPub]: https://www.w3.org/TR/activitypub/
[ActivityPubAuth]: https://www.w3.org/wiki/SocialCG/ActivityPub/Authentication_Authorization

## Copyright

CC0 1.0 Universal (CC0 1.0) Public Domain Dedication

To the extent possible under law, the authors of this Fediverse Enhancement Proposal have waived all copyright and related or neighboring rights to this work.
