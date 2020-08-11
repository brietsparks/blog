---
title: Context-bounded user stores
date: "2019-12-23T03:23:19Z"
description: Storing business-domain user data separate from a third party identity-provider 
---

My development team faced a decision of whether to keep all our user data in a third-party user identity-provider (e.g. Cognito, Auth0, Firebase), or split the user data between a users table in our bussiness-domain relational database and the identity provider.  At the end of the day we went with the latter, and here are a few reasons why.

The first and foremost reason is queryability. Consider the need to `getUsersByX` where `X` is anything other than the primary key. The split approach requires querying a single location that is the application database. Contrarily, with the identity-provider-only approach, this retrieval requires reading from two locations: 1) query that app database for user id's by `X` (where `X` is a resource which user id is attached to), and then 2) retrieve those users from the identity-service. If the identity-service supports batch user retrieval, then the entire retrieval process takes two requests. If the identity-service only support retrieving users one at a time, then multiple singular-retrievals in parallel or close succession becomes an issue for performance, cost, and rate-limiting/throttling. With our case, the bottom line ended up being batch user retrieval, which was not supported the identity provider, IBM Cloud AppID.

The split approach separates domain from auth concerns. All domain concerns live in the application database, and all the auth concerns (usernames, passwords, tokens) live in the identity provider. With this setup, our user stores have [bounded context](https://martinfowler.com/bliki/BoundedContext.html), where each can change independently. If we switch to another identity provider, then we don't have to migrate user profiles and potentially reassign all their ids; we just have to create new auth accounts and attach the new auth ids to our application's user records. If we need make a sweeping change across all users in the business domain, we don't have to make any requests to the identity-provider's.

Lastly, a user table in the application database provides referential integrity between each user and its associated data. Without an application users table, data can be written against users that potentially don't exist.
