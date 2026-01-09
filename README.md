# ActivityPub API

## Authors

- Evan Prodromou (Social Web Foundation)

## Participate

- [Issue tracker](https://github.com/swicg/activitypub-api/issues)
- [Mailing list](https://lists.w3.org/Archives/Public/public-swicg/)

## Introduction

[ActivityPub](https://www.w3.org/TR/activitypub/) defines a "Social API" that lets applications act on behalf of a user with a social networking platform.

The API is defined in several sections of the specification:

- [Section 3: Objects](https://www.w3.org/TR/activitypub/#obj)
- [Section 4: Actors](https://www.w3.org/TR/activitypub/#actors)
- [Section 5: Collections](https://www.w3.org/TR/activitypub/#collections)
- [Section 6: Client to Server Interactions](https://www.w3.org/TR/activitypub/#client-to-server-interactions)

Sections 3, 4, and 5 define a *read-only* interface to social data. Section 6 defines a *write* interface to make changes to social data by creating new [Activity](https://www.w3.org/TR/activitystreams-core/#activities) objects and implementing their defined side effects.

## User-Facing Problem

The ActivityPub API provides an extremely flexible interface for building social applications. Because unlimited kinds of activities can be created by clients, novel applications can be built on top of the ActivityPub network.

Unfortunately, the ActivityPub API has not been widely implemented in its read-write form, even by social platforms that implement the ActivityPub federation protocol.

Instead, they often implement platform-specific APIs, with a narrow range of functionality. [Mastodon](https://joinmastodon.org/), for example, implements the [Mastodon API](https://docs.joinmastodon.org/client/intro/), which focuses primarily on microblogging. [Threads](https://threads.com/) implements the [Threads API](https://developers.facebook.com/docs/threads/). Many other ActivityPub implementers have APIs that clone the Mastodon API, to make it easier for users to adapt third-party Mastodon clients to their platforms.

Sections 3, 4 and 5 of the ActivityPub specification overlap with the federation protocol, such that many platforms that implement the federation protocol effectively have an read-only API for clients. This functionality is rarely adapted for client applications, though.

Section 6, for creating new activities, is not widely implemented, either by clients or by servers.

Consequently, using the ActivityPub API to drive innovation in social software development is disappointingly slow or non-existent. Client developers don't build on top of the ActivityPub API, because so few servers and actors support it. Server developers don't implement it because there are few to no clients.

For users, this results in narrowly-focused social platforms that implement only some subset of the social functionality they can experience elsewhere: microblogging, events, image-sharing, or videos. The server developers implement all the end-user functionality *and* the full federation protocol.

### Goals

The primary goal of this task force is to encourage an innovative ecosystem where client developers can build interesting and novel applications with the ActivityPub API, and server developers can provide unexpectedly interesting experiences for their users. The method is to clear the path for both client and server developers to implement the standard.

The following [user stories](https://en.wikipedia.org/wiki/User_story) describe important work for this task force. User stories are tracked in the task force GitHub repository as issues.

- [OAuth profile](https://github.com/swicg/activitypub-api/issues/1): "As an ActivityPub API client developer, I need a standard profile of OAuth to use for authorising my app to API servers, so I don't have to create custom code for every different API server."
- [Server features](https://github.com/swicg/activitypub-api/issues/2): "As an ActivityPub API server developer, I need a list of recommended features to implement, so that I can support API clients correctly."
- [Feature discovery tips](https://github.com/swicg/activitypub-api/issues/3) "As an ActivityPub API client developer, I want to know how best to discover API features like media upload or readable inboxes, so that I can have reliable discovery code in my client software and have reasonable fallback strategies if features are missing."
- [Rate limiting](https://github.com/swicg/activitypub-api/issues/4) "As an ActivityPub client developer, I want a reliable and standard way to discover rate limits in an ActivityPub API server, so I can avoid unexpected failures for my users."
- [CORS](https://github.com/swicg/activitypub-api/issues/5) "As a Web client developer, I want to have reliable expectations for CORS for the ActivityPub API, so that I can use API endpoints from the browser without going through a proxy server."
- [Media upload](https://github.com/swicg/activitypub-api/issues/6) "As an ActivityPub API client developer, I want to have a standard definition of the media upload endpoint, so that I can upload media to servers."
- [Generic versus specialised servers](https://github.com/swicg/activitypub-api/issues/7) "As an ActivityPub API client developer, I want to know if an API server will process and distribute arbitrary Activity objects or just a specific subset, so that I can know whether the server will work for my application."
- [Client libraries](https://github.com/swicg/activitypub-api/issues/8) "As an ActivityPub API client developer, I want a library to use in my preferred programming language, so I don't have to develop directly to the REST API specification with low-level HTTP interfaces."
- [Push delivery](https://github.com/swicg/activitypub-api/issues/9) "As an ActivityPub user, I want data pushed from the server to my client device, so I don't have to reload a collection just to see if there's anything new."
- [Remote object access](https://github.com/swicg/activitypub-api/issues/10) "As an ActivityPub client developer, I want a reliable method for accessing objects on remote servers with the user's authorization, so I can read private or followers-only data."
- [Collection membership](https://github.com/swicg/activitypub-api/issues/11) As an ActivityPub client developer, I want to have a way to check if an object is a member of a collection, so I don't have to load the whole collection to search for the object."
- [Determining ActivityPub API support](https://github.com/swicg/activitypub-api/issues/12) "As an ActivityPub API client developer, I want to have a reliable way to tell if an actor supports the ActivityPub API, so I can set my client to read-only mode if not."
- [Filter collection by member type](https://github.com/swicg/activitypub-api/issues/13) "As an ActivityPub API client developer, I want a way to filter a collection by the type of the members, so I can show the members that are relevant to my application."
- [Find activities related to a single object](https://github.com/swicg/activitypub-api/issues/14) "As an ActivityPub client developer, I want a fast way to filter a collection of activities by the id of the object property, so I can find activities related to a particular object."
- [Filter collection by type of the object property](https://github.com/swicg/activitypub-api/issues/15) "As an ActivityPub API client developer, I want to filter a collection by the type of the object, so I that I can make an application that specializes in a particular kind of object."
- [Fallback representations](https://github.com/swicg/activitypub-api/issues/16) "As an ActivityPub API client developer, I want rules for a fallback representation for unrecognized activity or object types, so I can show a stand-in in a stream or other representation."
- [Separate notifications and home feed](https://github.com/swicg/activitypub-api/issues/21) "As an ActivityPub API client developer, I want a 'home feed' collection for content-oriented incoming activities like Create, Question and Announce, so that I can show my users the most important content from their networks." "As an ActivityPub API client developer, I want a 'notifications' collection for less directly important incoming activities like Like, Follow and Update, so that my users can see what's happening without it crowding up the 'home feed'."
- [Content search](https://github.com/swicg/activitypub-api/issues/22) "As an ActivityPub API client developer, I want a content search endpoint, so that I can find matching content by keyword."
- [Actor search](https://github.com/swicg/activitypub-api/issues/23) "As an ActivityPub API user, I want to be able to search for actors by keyword, so I can find my friends, family, colleagues, neighbours, influencers or publications that I like."
- [Prefix search](https://github.com/swicg/activitypub-api/issues/24) "As an ActivityPub API user, I want to have a prefix search feature available, so I can mention someone partially and let my client autocomplete the rest of their address."
- [Domain blocking](https://github.com/swicg/activitypub-api/issues/25) "As an ActivityPub API user, I want to have a way to block an entire domain, so that I am not harassed by users on a poorly-moderated server."
- [Mute](https://github.com/swicg/activitypub-api/issues/26) "As an ActivityPub API user, I want to be able to mute another actor, so we can stay connected but I don't see their activities in my inbox or in other collections."
- [Actor identity in OAuth authorization flow](https://github.com/swicg/activitypub-api/issues/29) "As an ActivityPub API client developer, I want to get an actor identity that I can use at the end of the authorization flow, so I know which actor actually logged into my application."
- [Standard error messages](https://github.com/swicg/activitypub-api/issues/30) "As an ActivityPub API client developer, I want to have an agreed-upon format for error messages from the server, so I can give more context to my user about why their activity didn't work than just saying 'That didn't work'".
- [Manually approved followers](https://github.com/swicg/activitypub-api/issues/31) "As an ActivityPub API client user who manually approves followers, I want a list of my current pending incoming follow requests, so I can decide whether to accept, reject, or ignore them." "As an ActivityPub API client user, I want a list of my current pending outgoing follow requests, so I can decide whether to keep waiting or give up and undo them."
- [Actor Profile Edit](https://github.com/swicg/activitypub-api/issues/33) "As an ActivityPub client user I would like to be able to edit my Actor profile: changing the avatar, display name, description, etc"
- [Local Feed](https://github.com/swicg/activitypub-api/issues/34) "As an ActivityPub API client developer, I want a feed of public activities from other actors on the same server."
- [Moderation](https://github.com/swicg/activitypub-api/issues/35) "As an ActivityPub API client developer, I would like to access a servers moderation queue. Allowing users with appropriate permissions to act upon flagged accounts and objects"
- [Trending hashtags, posts, and links](https://github.com/swicg/activitypub-api/issues/36) "As an ActivityPub API client user, I would like to see which Hashtags, Posts, Links are trending on my server, and on the wider fediverse"
- [Account recommendations](https://github.com/swicg/activitypub-api/issues/38) "As an ActivityPub API client user, I would like to access a list of recommended accounts".
- [Account directory](https://github.com/swicg/activitypub-api/issues/39) "As an ActivityPub API client developer, I would like to access a complete list of (public) actors from a given server".

### Non-goals

- Define a new, incompatible API to compete with the one described in the ActivityPub recommendation
- Adapt another standard API to the ActivityPub network
- Standardize a platform API, like the Mastodon API, for general use

## User research

TBD

## Proposed Approach

The task force will create [community group reports](https://www.w3.org/community/reports/) for each of the user stories in the goals. Some of these may be combined into a single report.

### Dependencies on non-stable features

N/A

### Solving OAuth profile with this approach

The group will create a report defining the expected flows and discovery mechanisms for OAuth 2.0.

There is an [example Web client](https://swicg.github.io/activitypub-api/examples/oauth/index.html) that can be used to test server implementations.

### Solving Push Delivery with this approach

There is a proposed report for push delivery: [Server-Sent Events For the ActivityPub API](https://swicg.github.io/activitypub-api/sse).

## Alternatives considered

TBD

## Accessibility, Internationalization, Privacy, and Security Considerations

TBD

## Stakeholder Feedback / Opposition

TBD

## References & acknowledgements

The following platforms inspire this approach:

- [Facebook Platform](https://en.wikipedia.org/wiki/Facebook_Platform)
- [OpenSocial](https://en.wikipedia.org/wiki/OpenSocial), in particular the [Activity Streams and Embedded Experiences API](https://www.w3.org/submissions/osapi/)
- [ATProto](https://atproto.com)'s use of [lexicons](https://atproto.com/guides/lexicon)