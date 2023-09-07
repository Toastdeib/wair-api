## Table of Contents

- [API Documentation](#api-documentation)
    - [Account APIs](#account-apis)
    - [Events APIs](#events-apis)
    - [Friend List APIs](#friend-list-apis)
    - [Ignore List APIs](#ignore-list-apis)
    - [Messages APIs](#messages-apis)
    - [Location APIs](#location-apis)
    - [Constants](#constants)
- [Glossary](#glossary)

# API Documentation

## Account APIs

#### /api/v0/accounts (POST)

Creates a new account with the provided username and password. **Note**: Usernames are case-insensitive and are restricted to alphanumeric characters and underscores.

##### Required headers:

- `Authorization` - Authentication header following the [Basic scheme](https://www.rfc-editor.org/rfc/rfc7617).
- `Platform` - A string indicating the platform this request is coming from. Can be one of:
    - `android`
    - `ios`
**Note**: The `Platform` header is technically optional and will be populated as `none` if omitted or invalid, but it's necessary to provide for push notification functionality.

##### Response payload:

```json
{
    "id": "user ID",
    "token": "session token"
}
```

`id` is the newly-created user's ID and `token` should be used in an `Authorization` header for authenticated API calls.

##### Possible error responses:

- HTTP 400 (BAD REQUEST) - Returned if the authentication header is omitted or malformed, or if the username is already in use.
- HTTP 500 (SERVER ERROR) - Returned if a database error occurs.

#### /api/v0/accounts/session (POST)

Logs into an existing account with the provided username and password. **Note**: Usernames are case-insensitive and are restricted to alphanumeric characters and underscores.

##### Required headers:

- `Authorization` - Authentication header following the [Basic scheme](https://www.rfc-editor.org/rfc/rfc7617).
- `Platform` - A string indicating the platform this request is coming from. Can be one of:
    - `android`
    - `ios`
**Note**: The `Platform` header is technically optional and will be populated as `none` if omitted or invalid, but it's necessary to provide for push notification functionality.

##### Response payload:

```json
{
    "id": "user ID",
    "token": "session token"
}
```

`id` is the logged-in user's ID and `token` should be used in an `Authorization` header for authenticated API calls.

##### Possible error responses:

- HTTP 400 (BAD REQUEST) - Returned if the authentication header is omitted or malformed, or if the username doesn't exist.
- HTTP 401 (UNAUTHORIZED) - Returned if the provided credentials are invalid.
- HTTP 500 (SERVER ERROR) - Returned if a database error occurs.

#### /api/v0/accounts/:id (GET)

Fetches an information payload for the provided user ID, if the provided token is valid for that user. If the token is invalid or belongs to a different user, the request will be rejected.

##### Required headers:

- `Authorization` - Authentication header following the [Bearer scheme](https://www.rfc-editor.org/rfc/rfc6750#section-2.1).

##### Response payload:

```json
{
    "id": "user ID",
    "displayName": "display name",
    "phoneNumber": "+11234567890",
    "emailAddress": "email.address@gmail.com",
    "privacyMode": false,
}
```

##### Possible error responses:

- HTTP 400 (BAD REQUEST) - Returned if the authentication header is omitted or malformed.
- HTTP 401 (UNAUTHORIZED) - Returned if the authentication token is expired or for the wrong user.
- HTTP 404 (NOT FOUND) - Returned if the provided ID doesn't exist in the database.
- HTTP 500 (SERVER ERROR) - Returned if a database error occurs.

#### /api/v0/accounts/:id (PUT)

Updates some or all of the personal information associated with the provider user ID, if the provided token is valid for that user. If the token is invalid or belongs to a different user, the request will be rejected. All body params are optional, and the response payload will only echo back those which were changed. If a field is omitted or unchanged from the existing value, it will be omitted from the response.

##### Required headers:

- `Authorization` - Authentication header following the [Bearer scheme](https://www.rfc-editor.org/rfc/rfc6750#section-2.1).

##### Body params:

- `displayName` - The new display name.
- `phoneNumber` - The new phone number, with country code.
- `emailAddress` - The new email address.
- `privacyMode` - A flag indicating whether the user should be in [privacy mode](#privacy-mode).

##### Response payload:

```json
{
    "id": "user ID",
    "displayName": "display name",
    "phoneNumber": "123-456-7890",
    "emailAddress": "email.address@gmail.com",
    "privacyMode": false,
}
```

##### Possible error responses:

- HTTP 400 (BAD REQUEST) - Returned if the authentication header is omitted or malformed.
- HTTP 401 (UNAUTHORIZED) - Returned if the authentication token is expired or for the wrong user.
- HTTP 404 (NOT FOUND) - Returned if the provided ID doesn't exist in the database.
- HTTP 500 (SERVER ERROR) - Returned if a database error occurs.

#### /api/v0/accounts/:id (DELETE)

Deletes the account for the provided user ID, if the provided token and credentials are valid for that user. If the token is invalid or belongs to a different user, or the credentials are invalid, the request will be rejected. **Note**: Any implementing clients should be sure to confirm with the user before sending this request, as the operation **cannot be reversed**. Terminated accounts will be deactivated and **all** personal information, including contact info, location data, events, and message history will be removed.

##### Required headers:

- `Authorization` - Authentication header following the [Bearer scheme](https://www.rfc-editor.org/rfc/rfc6750#section-2.1).
- `Verification` - Authentication header following the [Basic scheme](https://www.rfc-editor.org/rfc/rfc7617). **Note**: This header uses a non-standard name because this endpoint needs to support both token auth *and* credential-based auth for an extra layer of security.

##### Response payload:

```json
{}
```

##### Possible error responses:

- HTTP 400 (BAD REQUEST) - Returned if the authentication header is omitted or malformed.
- HTTP 401 (UNAUTHORIZED) - Returned if the authentication token is expired or for the wrong user.
- HTTP 404 (NOT FOUND) - Returned if the provided ID doesn't exist in the database.
- HTTP 500 (SERVER ERROR) - Returned if a database error occurs.

#### /api/v0/accounts/:id/session (DELETE)

Logs out and ends the session for the provided user ID, if the provided token is valid for that user. If the token is invalid or belongs to a different user, the request will be rejected.

##### Required headers:

- `Authorization` - Authentication header following the [Bearer scheme](https://www.rfc-editor.org/rfc/rfc6750#section-2.1).

##### Response payload:

```json
{}
```

##### Possible error responses:

- HTTP 400 (BAD REQUEST) - Returned if the authentication header is omitted or malformed.
- HTTP 401 (UNAUTHORIZED) - Returned if the authentication token is expired or for the wrong user.
- HTTP 404 (NOT FOUND) - Returned if the provided ID doesn't exist in the database.
- HTTP 500 (SERVER ERROR) - Returned if a database error occurs.

[Back to top](#table-of-contents)

## Events APIs

**Note**: All API calls under the `/events` path require a valid session.

#### /api/v0/events (GET)

Fetches a list of all current and future events, optionally filtering them down via the query string params.

##### Required headers:

- `Authorization` - Authentication header following the [Bearer scheme](https://www.rfc-editor.org/rfc/rfc6750#section-2.1).

##### Query params:

- `onlyMine` - An optional flag indicating whether the response should only contain events the provided user is an attendee of. If omitted, this will default to `false`.
- `search` - An optional field allowing the user to filter events by event name. If included, the value will be surrounded by wildcards in the resulting db query (i.e. `'%{search term}%'`), so it will find matches on any part of the event name.

##### Response payload:

```json
{
    "events": [
        {
            "id": "event ID",
            "name": "event name",
            "startDateUtc": "1970-01-01T00:00:00.000Z",
            "endDateUtc": "1970-01-01T00:00:00.000Z",
            "attendeeCount": 1,
            "requiresApproval": false
        },
        ...
    ]
}
```

##### Possible error responses:

- HTTP 400 (BAD REQUEST) - Returned if the authentication header is omitted or malformed.
- HTTP 401 (UNAUTHORIZED) - Returned if the authentication token is expired.
- HTTP 500 (SERVER ERROR) - Returned if a database error occurs.

#### /api/v0/events (POST)

Creates a new event with the provided event details. If any non-optional params are omitted, the request will fail.

##### Required headers:

- `Authorization` - Authentication header following the [Bearer scheme](https://www.rfc-editor.org/rfc/rfc6750#section-2.1).

##### Body params:

- `eventName` - The event name.
- `password` - The password required to join the event.
- `startDateUtc` - The start date for the event in UTC.
- `endDateUtc` - The end date for the event in UTC.
- `requiresApproval` - An optional flag indicating whether the event requires owner approval in addition to the password for anyone attempting to join it. If omitted, this will default to `false`.

##### Response payload:

```json
{
    "id": "event ID",
    "name": "event name",
    "startDateUtc": "1970-01-01T00:00:00.000Z",
    "endDateUtc": "1970-01-01T00:00:00.000Z",
    "requiresApproval": false
}
```

##### Possible error responses:

- HTTP 400 (BAD REQUEST) - Returned if the authentication header is omitted or malformed, or if the request body is missing required fields.
- HTTP 401 (UNAUTHORIZED) - Returned if the authentication token is expired.
- HTTP 500 (SERVER ERROR) - Returned if a database error occurs.

#### /api/v0/events/:id (GET)

Fetches the details for the provided event. If the event ID is invalid, the request will fail.

##### Required headers:

- `Authorization` - Authentication header following the [Bearer scheme](https://www.rfc-editor.org/rfc/rfc6750#section-2.1).

##### Response payload:

```json
{
    "id": "event ID",
    "name": "event name",
    "startDateUtc": "1970-01-01T00:00:00.000Z",
    "endDateUtc": "1970-01-01T00:00:00.000Z",
    "attendees": [
        {
            "id": "user ID",
            "displayName": "display name",
            "approved": false
        },
        ...
    ],
    "requiresApproval": false
}
```

**Note**: The `approved` field will only show up on attendee blobs if the logged-in user is the event owner **and** the event has its `requiresApproval` flag set to `true`. This flag should be used to present event owners with a UI control to approve any attendees whose `approved` flag is set to `false`.

##### Possible error responses:

- HTTP 400 (BAD REQUEST) - Returned if the authentication header is omitted or malformed.
- HTTP 401 (UNAUTHORIZED) - Returned if the authentication token is expired.
- HTTP 404 (NOT FOUND) - Returned if the event ID is invalid.
- HTTP 500 (SERVER ERROR) - Returned if a database error occurs.

#### /api/v0/events/:id (POST)

Joins an event as the logged-in user. If the event ID or event password are invalid, the request will fail.

##### Required headers:

- `Authorization` - Authentication header following the [Bearer scheme](https://www.rfc-editor.org/rfc/rfc6750#section-2.1).

##### Body params:

- `password` - The password required to join the event.

##### Response payload:

```json
{
    "id": "event ID",
    "name": "event name",
    "startDateUtc": "1970-01-01T00:00:00.000Z",
    "endDateUtc": "1970-01-01T00:00:00.000Z",
    "attendees": [
        {
            "id": "user ID",
            "displayName": "display name"
        },
        ...
    ],
    "requiresApproval": false,
    "approved": false
}
```

**Note**: The `approved` field will only show up if the event has its `requiresApproval` flag set to `true`, and indicates whether the logged-in user has been approved as an attendee by the event owner. Additionally, the `attendees` array will be omitted from this response in this case until the user is approved.

##### Possible error responses:

- HTTP 400 (BAD REQUEST) - Returned if the authentication header is omitted or malformed.
- HTTP 401 (UNAUTHORIZED) - Returned if the authentication token is expired or if the event password is invalid.
- HTTP 404 (NOT FOUND) - Returned if the event ID is invalid.
- HTTP 500 (SERVER ERROR) - Returned if a database error occurs.

#### /api/v0/events/:id (PUT)

Updates some or all of the event details, as specified by the body params. Similar to the `/accounts/:id` PUT path, the response will only include fields that actually change. If the event ID is invalid or the logged-in user isn't the event owner, the request will fail. The PUT verb will also be disabled on this path once an event starts. **Note**: The event password is immutable once the event is created, so the field will be ignored if included in the request body.

##### Required headers:

- `Authorization` - Authentication header following the [Bearer scheme](https://www.rfc-editor.org/rfc/rfc6750#section-2.1).

##### Body params:

- `eventName` - The event name.
- `startDateUtc` - The start date for the event in UTC.
- `endDateUtc` - The end date for the event in UTC.
- `requiresApproval` - An optional flag indicating whether the event requires owner approval in addition to the password for anyone attempting to join it.

##### Response payload:

```json
{
    "id": "event ID",
    "name": "event name",
    "startDateUtc": "1970-01-01T00:00:00.000Z",
    "endDateUtc": "1970-01-01T00:00:00.000Z",
    "requiresApproval": false
}
```

##### Possible error responses:

- HTTP 400 (BAD REQUEST) - Returned if the authentication header is omitted or malformed.
- HTTP 401 (UNAUTHORIZED) - Returned if the authentication token is expired or if the requesting user isn't the event owner.
- HTTP 404 (NOT FOUND) - Returned if the event ID is invalid.
- HTTP 405 (METHOD NOT ALLOWED) - Returned if the current timestamp is later than the event's start date.
- HTTP 500 (SERVER ERROR) - Returned if a database error occurs.

#### /api/v0/events/:id (DELETE)

Deletes the event, removing all attendees from it in the process. If the event ID is invalid or the logged-in user isn't the event owner, the request will fail. **Note**: Any implementing clients should be sure to confirm with the user before sending this request, as the operation **cannot be reversed**. 

##### Required headers:

- `Authorization` - Authentication header following the [Bearer scheme](https://www.rfc-editor.org/rfc/rfc6750#section-2.1).

##### Response payload:

```json
{}
```

##### Possible error responses:

- HTTP 400 (BAD REQUEST) - Returned if the authentication header is omitted or malformed.
- HTTP 401 (UNAUTHORIZED) - Returned if the authentication token is expired or if the requesting user isn't the event owner.
- HTTP 404 (NOT FOUND) - Returned if the event ID is invalid.
- HTTP 500 (SERVER ERROR) - Returned if a database error occurs.

#### /api/v0/events/:id/:userId (PUT)

Confirms the provided user ID as an attendee of the provided event. If the event ID or user ID are invalid, or the logged-in user isn't the event owner, the request will fail.

##### Required headers:

- `Authorization` - Authentication header following the [Bearer scheme](https://www.rfc-editor.org/rfc/rfc6750#section-2.1).

##### Body params:

- `approved` - A flag indicating whether the user is approved. If `true`, the provisional user will be added to the event as an attendee, and if `false`, the provisional user will be removed from the event. If the param is omitted, the request will fail.

##### Response payload:

```json
{}
```

##### Possible error responses:

- HTTP 400 (BAD REQUEST) - Returned if the authentication header is omitted or malformed, or the body param is omitted or invalid.
- HTTP 401 (UNAUTHORIZED) - Returned if the authentication token is expired or if the requesting user isn't the event owner.
- HTTP 404 (NOT FOUND) - Returned if the event ID is invalid.
- HTTP 500 (SERVER ERROR) - Returned if a database error occurs.

#### /api/v0/events/:id/:userId (DELETE)

Removes the provided attendee from the provided event. If the event ID is invalid or the logged-in user isn't the provided user ID or the event owner, the request will fail. **Note**: Any implementing clients should be sure to confirm with the user before sending this request, as the operation **cannot be reversed**. 

##### Required headers:

- `Authorization` - Authentication header following the [Bearer scheme](https://www.rfc-editor.org/rfc/rfc6750#section-2.1).

##### Response payload:

```json
{}
```

##### Possible error responses:

- HTTP 400 (BAD REQUEST) - Returned if the authentication header is omitted or malformed.
- HTTP 401 (UNAUTHORIZED) - Returned if the authentication token is expired, or if the requesting user isn't either the provided user ID or the event owner.
- HTTP 404 (NOT FOUND) - Returned if the event ID is invalid.
- HTTP 500 (SERVER ERROR) - Returned if a database error occurs.

[Back to top](#table-of-contents)

## Friend List APIs

**Note**: All API calls under the `/friendlist` path require a valid session.

#### /api/v0/friendlist (GET)

Fetches all users on the logged-in user's friend list. Open requests that haven't been confirmed by the other party yet will be marked as not approved.

##### Required headers:

- `Authorization` - Authentication header following the [Bearer scheme](https://www.rfc-editor.org/rfc/rfc6750#section-2.1).

##### Response payload:

```json
{
    "friendList": [
        {
            "id": "friend ID",
            "displayName": "display name",
            "approved": false,
        },
        ...
    ]
}
```

##### Possible error responses:

- HTTP 400 (BAD REQUEST) - Returned if the authentication header is omitted or malformed.
- HTTP 401 (UNAUTHORIZED) - Returned if the authentication token is expired.
- HTTP 500 (SERVER ERROR) - Returned if a database error occurs.

#### /api/v0/friendlist (POST)

Adds the provided user to the logged-in user's friend list. If this is the first request for a pair of IDs, the friend will be added provisionally, pending confirmation by the other party. The second request confirms it, fully adding both parties to each other's lists.

##### Required headers:

- `Authorization` - Authentication header following the [Bearer scheme](https://www.rfc-editor.org/rfc/rfc6750#section-2.1).

##### Body params:

- `friendId` - The user ID for the friend the user is attempting to add.

##### Response payload:

```json
{
    "id": "friend ID",
    "displayName": "display name",
    "approved": false,
}
```

The `approved` field will be returned as `false` if it's a provisional add and `true` if the request is confirming an existing provisional friend.

##### Possible error responses:

- HTTP 400 (BAD REQUEST) - Returned if the authentication header is omitted or malformed, or if the body param is omitted or invalid.
- HTTP 401 (UNAUTHORIZED) - Returned if the authentication token is expired.
- HTTP 500 (SERVER ERROR) - Returned if a database error occurs.

#### /api/v0/friendlist/:id (GET)

Fetches the full friend details for the provided user ID. If the ID doesn't exist on the logged-in user's friend list, the request will fail.

##### Required headers:

- `Authorization` - Authentication header following the [Bearer scheme](https://www.rfc-editor.org/rfc/rfc6750#section-2.1).

##### Response payload:

```json
{
    "id": "friend ID",
    "displayName": "display name",
    "approved": false,
    "notes": "user-defined notes"
}
```

##### Possible error responses:

- HTTP 400 (BAD REQUEST) - Returned if the authentication header is omitted or malformed.
- HTTP 401 (UNAUTHORIZED) - Returned if the authentication token is expired.
- HTTP 404 (NOT FOUND) - Returned if the provided ID doesn't exist or isn't on the logged-in user's friend list.
- HTTP 500 (SERVER ERROR) - Returned if a database error occurs.

#### /api/v0/friendlist/:id (PUT)

Updates the personal friend list details for the provided user ID. If the ID doesn't exist on the logged-in user's friend list, the request will fail. Similar to the `/accounts/:id` PUT path, the response will only include fields that actually change.

##### Required headers:

- `Authorization` - Authentication header following the [Bearer scheme](https://www.rfc-editor.org/rfc/rfc6750#section-2.1).

##### Body params:

- `notes` - User-defined notes for a friend, for saving any desired information about them. This has a character limit of 300.

##### Response payload:

```json
{
    "id": "friend ID",
    "notes": "user-defined notes"
}
```

##### Possible error responses:

- HTTP 400 (BAD REQUEST) - Returned if the authentication header is omitted or malformed.
- HTTP 401 (UNAUTHORIZED) - Returned if the authentication token is expired.
- HTTP 404 (NOT FOUND) - Returned if the provided ID doesn't exist or isn't on the logged-in user's friend list.
- HTTP 500 (SERVER ERROR) - Returned if a database error occurs.

#### /api/v0/friendlist/:id (DELETE)

Removes the provided user ID from the logged-in user's friend list. If the ID doesn't exist on the logged-in user's friend list, the request will fail. This operation is mutually destructive - if a user removes someone from their friend list, they'll be removed from the friend's list as well. **Note**: Any implementing clients should be sure to confirm with the user before sending this request, as the operation **cannot be reversed**.

As currently specced out, this operation will **also** destroy all message history between the two users in question. That may be changed in the future to archive message history instead of deleting it outright.

##### Required headers:

- `Authorization` - Authentication header following the [Bearer scheme](https://www.rfc-editor.org/rfc/rfc6750#section-2.1).

##### Response payload:

```json
{}
```

##### Possible error responses:

- HTTP 400 (BAD REQUEST) - Returned if the authentication header is omitted or malformed.
- HTTP 401 (UNAUTHORIZED) - Returned if the authentication token is expired.
- HTTP 404 (NOT FOUND) - Returned if the provided ID doesn't exist or isn't on the logged-in user's friend list.
- HTTP 500 (SERVER ERROR) - Returned if a database error occurs.

[Back to top](#table-of-contents)

## Ignore List APIs

**Note**: All API calls under the `/ignorelist` path require a valid session.

#### /api/v0/ignorelist (GET)

Fetches all users on the logged-in user's ignore list.

##### Required headers:

- `Authorization` - Authentication header following the [Bearer scheme](https://www.rfc-editor.org/rfc/rfc6750#section-2.1).

##### Response payload:

```json
{
    "ignoreList": [
        {
            "id": "ignored user ID",
            "displayName": "display name",
        },
        ...
    ]
}
```

##### Possible error responses:

- HTTP 400 (BAD REQUEST) - Returned if the authentication header is omitted or malformed.
- HTTP 401 (UNAUTHORIZED) - Returned if the authentication token is expired.
- HTTP 500 (SERVER ERROR) - Returned if a database error occurs.

#### /api/v0/ignorelist (POST)

Adds the provided user to the logged-in user's ignore list.

##### Required headers:

- `Authorization` - Authentication header following the [Bearer scheme](https://www.rfc-editor.org/rfc/rfc6750#section-2.1).

##### Body params:

- `ignoreId` - The user ID for the user the logged-in user is attempting to add.

##### Response payload:

```json
{
    "id": "ignored user ID",
    "displayName": "display name"
}
```

##### Possible error responses:

- HTTP 400 (BAD REQUEST) - Returned if the authentication header is omitted or malformed, or if the body param is omitted or invalid.
- HTTP 401 (UNAUTHORIZED) - Returned if the authentication token is expired.
- HTTP 500 (SERVER ERROR) - Returned if a database error occurs.

#### /api/v0/ignorelist/:id (GET)

Fetches the full ignore list details for the provided user ID. If the ID doesn't exist on the logged-in user's ignore list, the request will fail.

##### Required headers:

- `Authorization` - Authentication header following the [Bearer scheme](https://www.rfc-editor.org/rfc/rfc6750#section-2.1).

##### Response payload:

```json
{
    "id": "ignored user ID",
    "displayName": "display name",
    "notes": "user-defined notes"
}
```

##### Possible error responses:

- HTTP 400 (BAD REQUEST) - Returned if the authentication header is omitted or malformed.
- HTTP 401 (UNAUTHORIZED) - Returned if the authentication token is expired.
- HTTP 404 (NOT FOUND) - Returned if the provided ID doesn't exist or isn't on the logged-in user's ignore list.
- HTTP 500 (SERVER ERROR) - Returned if a database error occurs.

#### /api/v0/ignorelist/:id (PUT)

Updates the personal ignore list details for the provided user ID. If the ID doesn't exist on the logged-in user's ignore list, the request will fail. Similar to the `/accounts/:id` PUT path, the response will only include fields that actually change.

##### Required headers:

- `Authorization` - Authentication header following the [Bearer scheme](https://www.rfc-editor.org/rfc/rfc6750#section-2.1).

##### Body params:

- `notes` - User-defined notes for an ignored user, for saving any desired information about them. This has a character limit of 300.

##### Response payload:

```json
{
    "id": "ignored user ID",
    "notes": "user-defined notes"
}
```

##### Possible error responses:

- HTTP 400 (BAD REQUEST) - Returned if the authentication header is omitted or malformed.
- HTTP 401 (UNAUTHORIZED) - Returned if the authentication token is expired.
- HTTP 404 (NOT FOUND) - Returned if the provided ID doesn't exist or isn't on the logged-in user's ignore list.
- HTTP 500 (SERVER ERROR) - Returned if a database error occurs.

#### /api/v0/ignorelist/:id (DELETE)

Removes the provided user ID from the logged-in user's ignore list. If the ID doesn't exist on the logged-in user's ignore list, the request will fail. **Note**: Any implementing clients should be sure to confirm with the user before sending this request, as the operation **cannot be reversed**.

##### Required headers:

- `Authorization` - Authentication header following the [Bearer scheme](https://www.rfc-editor.org/rfc/rfc6750#section-2.1).

##### Response payload:

```json
{}
```

##### Possible error responses:

- HTTP 400 (BAD REQUEST) - Returned if the authentication header is omitted or malformed.
- HTTP 401 (UNAUTHORIZED) - Returned if the authentication token is expired.
- HTTP 404 (NOT FOUND) - Returned if the provided ID doesn't exist or isn't on the logged-in user's ignore list.
- HTTP 500 (SERVER ERROR) - Returned if a database error occurs.

[Back to top](#table-of-contents)

## Messages APIs

**Note**: All API calls under the `/messages` path require a valid session.

#### /api/v0/messages (GET)

Fetches a list of message threads the user is a part of, including the most recent message sent on each thread. The number of threads returned and how far back the list should start can be specified by query params; by default, it will return the 15 most recent.

##### Required headers:

- `Authorization` - Authentication header following the [Bearer scheme](https://www.rfc-editor.org/rfc/rfc6750#section-2.1).

##### Query params:

- `threadCount` - The number of message threads that should be returned. If omitted, it will default to 15.
- `startAt` - How far back the list of returned message threads should start. This is **zero-indexed**. If omitted, it will default to 0.

##### Response payload:

```json
{
    "messageList": [
        {
            "id": "user ID of the other party",
            "displayName": "display name",
            "lastMessage": "last message",
            "timestampUtc": "1970-01-01T00:00:00.000Z"
        },
        ...
    ]
}
```

##### Possible error responses:

- HTTP 400 (BAD REQUEST) - Returned if the authentication header is omitted or malformed.
- HTTP 401 (UNAUTHORIZED) - Returned if the authentication token is expired.
- HTTP 500 (SERVER ERROR) - Returned if a database error occurs.

#### /api/v0/messages/:id (GET)

Fetches the logged-in user's message history for the provided user ID. The number of messages returned and how far back the list should start can be specified by query params; by default, it will return the 50 most recent.

##### Required headers:

- `Authorization` - Authentication header following the [Bearer scheme](https://www.rfc-editor.org/rfc/rfc6750#section-2.1).

##### Query params:

- `messageCount` - The number of messages that should be returned. If omitted, it will default to 50.
- `startAt` - How far back the list of returned messages should start. This is **zero-indexed**. If omitted, it will default to 0.

##### Response payload:

```json
{
    "messages": [
        {
            "id": "message ID",
            "senderId": "user ID for the message's sender"
            "displayName": "display name for the message's sender",
            "body": "message body",
            "timestampUtc": "1970-01-01T00:00:00.000Z",
            "edited": false
        },
        ...
    ]
}
```

##### Possible error responses:

- HTTP 400 (BAD REQUEST) - Returned if the authentication header is omitted or malformed.
- HTTP 401 (UNAUTHORIZED) - Returned if the authentication token is expired.
- HTTP 404 (NOT FOUND) - Returned if the provided user ID doesn't exist or isn't on the logged-in user's friend list.
- HTTP 500 (SERVER ERROR) - Returned if a database error occurs.

#### /api/v0/messages/:id (POST)

Sends a new message to the provided user ID. If the provided user ID is invalid or no message is included in the request body, the request will fail.

##### Required headers:

- `Authorization` - Authentication header following the [Bearer scheme](https://www.rfc-editor.org/rfc/rfc6750#section-2.1).

##### Body params:

- `message` - The message to send. This has a character limit of 300.

##### Response payload:

```json
{
    "id": "message ID",
    "senderId": "user ID for the message's sender"
    "displayName": "display name for the message's sender",
    "body": "message body",
    "timestampUtc": "1970-01-01T00:00:00.000Z"
}
```

##### Possible error responses:

- HTTP 400 (BAD REQUEST) - Returned if the authentication header is omitted or malformed.
- HTTP 401 (UNAUTHORIZED) - Returned if the authentication token is expired.
- HTTP 404 (NOT FOUND) - Returned if the provided user ID doesn't exist or isn't on the logged-in user's friend list.
- HTTP 500 (SERVER ERROR) - Returned if a database error occurs.

#### /api/v0/messages/:messageId (PUT)

Edits the body of the provided message ID. If the message ID doesn't exist, the logged-in user doesn't own the message, or no message is included in the request body, the request will fail.

##### Required headers:

- `Authorization` - Authentication header following the [Bearer scheme](https://www.rfc-editor.org/rfc/rfc6750#section-2.1).

##### Body params:

- `message` - The updated message body.

##### Response payload:

```json
{
    "id": "message ID",
    "body": "message body",
    "timestampUtc": "1970-01-01T00:00:00.000Z",
    "edited": true
}
```

##### Possible error responses:

- HTTP 400 (BAD REQUEST) - Returned if the authentication header is omitted or malformed.
- HTTP 401 (UNAUTHORIZED) - Returned if the authentication token is expired or if the message doesn't belong to the logged-in user.
- HTTP 404 (NOT FOUND) - Returned if the provided message ID doesn't exist.
- HTTP 500 (SERVER ERROR) - Returned if a database error occurs.

#### /api/v0/messages/:messageId (DELETE)

Deletes the provided message ID. If the message ID doesn't exist or the logged-in user doesn't own the message, the request will fail. **Note**: Any implementing clients should be sure to confirm with the user before sending this request, as the operation **cannot be reversed**.

##### Required headers:

- `Authorization` - Authentication header following the [Bearer scheme](https://www.rfc-editor.org/rfc/rfc6750#section-2.1).

##### Response payload:

```json
{}
```

##### Possible error responses:

- HTTP 400 (BAD REQUEST) - Returned if the authentication header is omitted or malformed.
- HTTP 401 (UNAUTHORIZED) - Returned if the authentication token is expired or if the message doesn't belong to the logged-in user.
- HTTP 404 (NOT FOUND) - Returned if the provided message ID doesn't exist.
- HTTP 500 (SERVER ERROR) - Returned if a database error occurs.

[Back to top](#table-of-contents)

## Location APIs

**Note**: All API calls under the `/location` path require a valid session.

#### /api/v0/location/:id (PUT)

Sets the current location of given user ID to the provided latitude/longitude pair. If either field is omitted or the request is for a user ID that isn't the logged-in user, the request will fail.

##### Required headers:

- `Authorization` - Authentication header following the [Bearer scheme](https://www.rfc-editor.org/rfc/rfc6750#section-2.1).

##### Body params:

- `latitude` - The user's current latitude.
- `longitude` - The user's current longitude.

##### Response payload:

```json
{
    "id": "user ID",
    "latitude": "latitude",
    "longitude": "longitude",
    "timestampUtc": "1970-01-01T00:00:00.000Z"
}
```

##### Possible error responses:

- HTTP 400 (BAD REQUEST) - Returned if the authentication header is omitted or malformed.
- HTTP 401 (UNAUTHORIZED) - Returned if the authentication token is expired or for the wrong user.
- HTTP 404 (NOT FOUND) - Returned if the provided ID doesn't exist in the database.
- HTTP 500 (SERVER ERROR) - Returned if a database error occurs.

#### /api/v0/location/:id (DELETE)

Clears the current location of given user ID to the provided latitude/longitude pair. If the request is for a user ID that isn't the logged-in user, the request will fail.

##### Required headers:

- `Authorization` - Authentication header following the [Bearer scheme](https://www.rfc-editor.org/rfc/rfc6750#section-2.1).

##### Response payload:

```json
{}
```

##### Possible error responses:

- HTTP 400 (BAD REQUEST) - Returned if the authentication header is omitted or malformed.
- HTTP 401 (UNAUTHORIZED) - Returned if the authentication token is expired or for the wrong user.
- HTTP 404 (NOT FOUND) - Returned if the provided ID doesn't exist in the database.
- HTTP 500 (SERVER ERROR) - Returned if a database error occurs.

#### /api/v0/location/:eventId (GET)

Fetches location data for all attendees of the provided event ID who aren't in [privacy mode](#privacy-mode). If the event ID doesn't exist, the logged-in user isn't an attendee, or the logged-in user is in privacy mode themselves, the request will fail.

##### Required headers:

- `Authorization` - Authentication header following the [Bearer scheme](https://www.rfc-editor.org/rfc/rfc6750#section-2.1).

##### Response payload:

```json
{
    "id": "event ID",
    "locations": [
        {
            "id": "user ID",
            "displayName": "display name"
            "latitude": "latitude",
            "longitude": "longitude",
            "timestampUtc": "1970-01-01T00:00:00.000Z"
        },
        ...
    ]
}
```

##### Possible error responses:

- HTTP 400 (BAD REQUEST) - Returned if the authentication header is omitted or malformed, or if the user is in privacy mode.
- HTTP 401 (UNAUTHORIZED) - Returned if the authentication token is expired or if the user isn't an attendee for the event.
- HTTP 404 (NOT FOUND) - Returned if the provided ID doesn't exist in the database.
- HTTP 500 (SERVER ERROR) - Returned if a database error occurs.

## Constants

TBD

[Back to top](#table-of-contents)

# Glossary

##### Privacy mode

"Privacy mode" in the context of Wair is a user deciding to temporarily remove themselves from *all* of their events without actually doing so. While privacy mode is enabled, a user's location will not be updated or shared with other attendees of their active events, and they will be unable to view the locations of other users in their active events. When toggled back off, location updating and sharing will resume and the user will once again be able to view the locations of others as well.

[Back to top](#table-of-contents)
