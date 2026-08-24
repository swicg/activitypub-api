---
slug: "c180"
authors: Evan Prodromou <evan@socialwebfoundation.org>
status: DRAFT
discussionsTo: https://codeberg.org/evanp/fep/issues
dateReceived: 2025-03-11
trackingIssue: https://codeberg.org/fediverse/fep/issues/531
---
# FEP-c180: Problem Details for ActivityPub


## Summary

ActivityPub is a RESTful API and HTTP-based protocol for standards-based social networking, but does not specify an error format. This document provides a profile of the Problem Details for HTTP APIs specification ([RFC 9457][RFC 9457]) for use with ActivityPub.

## Introduction

ActivityPub is the W3C standard for federated social networking. It describes a standard RESTful API for social applications that allows people to create and share social content like text, images, audio and video, as well as reacting to social content and building a social graph of connections between people. ActivityPub also includes a standard protocol for federating social content between servers, so that people on different social platforms can interact with each other.

Both the client API and the server-to-server protocol are based on HTTP, and use HTTP status codes to indicate the success or failure of requests. However, HTTP status codes are not always sufficient to describe the nature of an error, or to provide enough information for a client to recover from an error.

The Problem Details for HTTP APIs specification ([RFC 9457][RFC 9457]) describes a way to provide more detailed information about errors in an HTTP response. The format includes a machine-readable description of the error, as well as a human-readable explanation, additional data about the error, and a link to more information about the error.

This document describes a number of specific error types that are relevant to ActivityPub, and provides guidance on how to use the Problem Details for HTTP APIs format with ActivityPub.

## Motivating use cases

- As an ActivityPub API client developer, I want a machine-readable description of an error from the server, so that I can provide actionable and localised error messages to the user.
- As an ActivityPub API client developer, I want to know if I can recover from an erroneous request, so that I can provide a more robust experience.
- As a server developer, I want to know if I can recover from an erroneous request, so that I can provide more reliable communications.
- As a server developer, I want to know if a remote server would ever accept the activity I am sending, so that I can save resources by not sending activities that will be rejected.

## Specification

ActivityPub servers SHOULD use the Problem Details for HTTP APIs format to describe errors in responses to HTTP requests. The format is described in [RFC 9457][RFC 9457].

These types of HTTP request in the ActivityPub API and federation protocol SHOULD use the Problem Details format for errors (abbreviations used in this document are in parentheses):

- GET requests for objects, including activities, actors, and collections ("get")
- POST requests to the outbox of an actor, for the creation of new activities ("outbox")
- POST requests for [media uploads][Media] ("media upload")
- POST requests for proxy URLs ("proxy")
- POST requests to the inbox of an actor, for the delivery of remote activities ("inbox")
- POST requests to the shared inbox for a collection of actors ("shared inbox")

Other ActivityPub requests MAY use the Problem Details format.

The `about:blank` type defined in [RFC 9457][RFC 9457] MAY be used for problems that do not have a specific type. Other types registered in the [IANA Problem Type Registry][Registry] MAY be used for specific problems.

### Problem types for ActivityPub

Problem types in this vocabulary use the https://w3id.org/fep/c180 prefix.

Each of the following problem types lists the applicability of the problem (per the list of request types above), the type URI, the title of the problem, the HTTP status code that SHOULD be used, and additional fields that MAY be included in the response.

#### Unsupported type

- applicability: inbox, outbox, sharedInbox, media upload
- type: https://w3id.org/fep/c180#unsupported-type
- title: Unsupported type
- status: 400 Bad Request
- additional fields:
  - id: The `id` of the object with the unsupported type
  - type: The `type` that is not supported

This indicates that the type of the activity, or one of the objects referred to by the activity, is not supported by the API server or the receiving federation protocol server.

#### Object does not exist

- applicability: inbox, outbox, sharedInbox, media upload
- type: https://w3id.org/fep/c180#object-does-not-exist
- title: Object does not exist
- status: 400 Bad Request
- additional fields:
  - id: The `id` of the object that does not exist

The activity refers to an object in one of its properties, such as `object`, `target`, or an addressing property, but the object does not exist. Recursively connected objects, like the `inReplyTo` property of the `object` property, can also be checked.

Note that this type is distinct from an endpoint returning a 404 Not Found status code for a GET request for an object that does not exist, or for posting to an endpoint that does not exist.

#### Duplicate delivery

- applicability: inbox, sharedInbox
- type: https://w3id.org/fep/c180#duplicate-delivery
- title: Duplicate delivery
- status: 400 Bad Request
- additional fields:
  - id: The `id` of the activity that was previously delivered

The activity has already been delivered to the `inbox` or to all accounts using the `sharedInbox`.

Note that this is different from [Redundant activity](#redundant-activity). Duplicate delivery is when the same activity is delivered multiple times. Redundant activity is when two different activities that
do the same thing are received.

#### Redundant activity

- applicability: inbox, outbox, sharedInbox, media upload
- type: https://w3id.org/fep/c180#redundant-activity
- title: Redundant activity
- status: 400 Bad Request
- additional fields:
  - duplicate: The `id` of the previous activity

The activity is a duplicate of a previous activity which has already been processed by the server, and which has not been reverted with an `Undo` activity or with other activities. The `duplicate` property contains the `id` of the previous activity.

Activity types that are often treated as idempotent and can only be processed once include `Create`, `Delete`, `Follow`, `Accept`, `Reject`, `Add`, `Remove`, `Block`, `Undo`, and `Like`. Other activity types like `Announce` are treated as idempotent by some servers.

Note that this is different from [Duplicate delivery](#duplicate-delivery).Redundant activity is when two different activities that do the same thing are received.  Duplicate delivery is when the same activity is delivered multiple times.

#### Approval required

- applicability: inbox, outbox, sharedInbox, media upload
- type: https://w3id.org/fep/c180#approval-required
- title: Approval required
- status: 202 Accepted
- additional fields:
  - approver: The `id` of the actor who must approve the activity

The activity will be delivered to the addressees, but may not have side effects applied until it is approved by an administrator, moderator, or one of the addressees.

For example, a `Follow` activity may be delivered to the addressee's inbox, but the `Accept` activity may not be returned until the addressee approves the follow request.

As another example, a `Create` activity with an `object` property with an `inReplyTo` property may require approval by the author of the replied-to object before it is added to that object's `replies` collection.

This problem type would be used for activities that are manually approved, not automatically approved.

#### Not an actor

- applicability: inbox, outbox, sharedInbox, media upload
- type: https://w3id.org/fep/c180#not-an-actor
- title: Not an actor
- status: 400 Bad Request
- additional fields:
  - id: The `id` of the object that is not an actor

The activity refers to an object in one of its properties, such as `object` or an addressing property, that requires an ActivityPub actor to be correctly processed, but the object is not an actor.

#### Principal-actor mismatch

- applicability: inbox, outbox, sharedInbox, media upload
- type: https://w3id.org/fep/c180#principal-actor-mismatch
- title: Principal-actor mismatch
- status: 400 Bad Request
- additional fields:
  - principal: The `id` of the principal
  - actor: The `id` of the actor

The security principal of the request, such as the authenticated user, does not match the actor that is the subject of the activity.

For example, the authenticated user is trying to send a `Follow` activity to another actor, but the `actor` property of the activity is not the authenticated user.

Note that it is possibly valid for the `actor` property of an activity to not be the same as the authenticated user; for example, with [inbox forwarding](https://www.w3.org/TR/activitypub/#inbox-forwarding).

#### Actor not authorized

- applicability: inbox, outbox, sharedInbox, media upload
- type: https://w3id.org/fep/c180#actor-not-authorized
- title: Actor not authorized
- status: 403 Forbidden
- additional fields:
  - actor: The `id` of the actor
  - resource: The `id` of the resource the actor is unauthorized to access

The actor is not authorized to perform the given activity on, to, or from a given object.

For example, with an `Add` activity, the actor is not authorized to add the `object` to the `target` collection.

As another example, with a `Delete` activity, the actor is not authorized to delete the `object`.

Another example would be a `Like` activity for an object where the actor has been blocked by the creator of the object.

Note that this type is distinct from [Principal not authorized](#principal-not-authorized), which indicates that the authenticated user is not authorized to perform the activity.

#### Principal not authorized

- applicability: inbox, outbox, sharedInbox, media upload, proxy, get
- type: https://w3id.org/fep/c180#principal-not-authorized
- title: Principal not authorized
- status: 403 Forbidden
- additional fields:
  - principal: The `id` of the principal
  - resource: The `id` of the resource the principal is unauthorized to access

This problem type indicates that the security principal, such as the authenticated user, is not authorized to perform the given activity on, to, or from a given object.

It can also be used to indicate that the authenticated user is not authorized to GET an object, either directly or through a proxy.

This type is distinct from [actor not authorized](#actor-not-authorized). This type should only be used when the principal and the actor are distinct, or when there is no actor (such as with GET requests).

#### Client not authorized

- applicability: outbox, media upload, proxy, get
- type: https://w3id.org/fep/c180#client-not-authorized
- title: Client not authorized
- status: 403 Forbidden
- additional fields:
  - client: The `id` of the client

This problem type is applicability GET and POST requests.

This indicates that the client is not authorized to perform the given activity. The security principal, like the authenticated user, may be authorized, but the client is not.

An example would be a client that uses OAuth 2.0 to authenticate, perhaps with [FEP-d8c2](https://w3id.org/fep/d8c2), but has not been granted the proper [scopes](https://oauth.net/2/scope/) to perform the activity.

This error type implies, but does not promise, that the security principal would be authorized to perform the activity with a different client.

This problem type is primarily for the ActivityPub API, between a client and a server.  In the case of the federation protocol, where the principal is closely tied to the platform that is sending the activity, there may not be a meaningful way for the principal to interact without the client (in this case, their server).

#### Unsupported media type

- applicability: media upload
- type: https://w3id.org/fep/c180#unsupported-media-type
- title: Unsupported media type
- status: 400 Bad Request
- additional fields:
  - filename: The filename of the media
  - mediaType: The unsupported media type

The media type of the uploaded file is not supported by the server.

#### Media too large

- applicability: media upload
- type: https://w3id.org/fep/c180#media-too-large
- title: Media too large
- status: 413 Payload Too Large
- additional fields:
  - filename: The filename of the media
  - size: The size of the media in bytes
  - maxSize: The maximum size of the media in bytes

The uploaded file is too large to be processed by the server.

#### No applicable addressees

- applicability: inbox, sharedInbox
- type: https://w3id.org/fep/c180#no-applicable-addressees
- title: No applicable addressees
- status: 400 Bad Request

The activity does not have any addressees that are applicable to the server. This could be because the activity has no `to`, `cc`, or `bcc` properties, or because the addressees do not have inboxes on the server.

Another case is where the addressees are [Collections](https://www.w3.org/TR/activitystreams-core#collections), and no actor in the collection has an inbox on the server. For example, if an activity is addressed to the actor's [followers](https://www.w3.org/TR/activitypub#followers) collection, but none of the followers have inboxes on the server.

#### Rate limit exceeded

- applicability: outbox, media upload, get, proxy, inbox, sharedInbox
- type: https://w3id.org/fep/c180#rate-limit-exceeded
- title: Rate limit exceeded
- status: 429 Too Many Requests

The client or the security principal has exceeded the rate limit for the given activity. The server MAY include a `Retry-After` header in the response to indicate when the rate limit will be reset.

This problem type is primarily applicable to the ActivityPub API, between a client and a server. It is unusual for a server to rate limit incoming activities over the federation protocol.

## Privacy considerations

Some of the problem types in this document may reveal information about the server's internal state, such as the existence of an object, the relationship of an object to an actor, or a relationship between actors. Servers should be careful to avoid revealing sensitive information in error messages.

## References

- Christine Lemmer Webber, Jessica Tallon, [ActivityPub][ActivityPub], 2018
- Christine Lemmer Webber, Amy Guy, et. al., [Media Upload][Media], 2018
- M. Nottingham, E. Wilde, S. Dalal, [RFC 9457: Problem Details for HTTP APIs][RFC 9457], 2023

[ActivityPub]: https://www.w3.org/TR/activitypub/
[RFC 9457]: https://www.rfc-editor.org/rfc/rfc9457.html
[Registry]: https://www.iana.org/assignments/problem-types/problem-types.xhtml
[Media]: https://www.w3.org/wiki/SocialCG/ActivityPub/MediaUpload

## Copyright

CC0 1.0 Universal (CC0 1.0) Public Domain Dedication

To the extent possible under law, the authors of this Fediverse Enhancement Proposal have waived all copyright and related or neighboring rights to this work.
