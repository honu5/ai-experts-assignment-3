## The Bug

The `Client.request()` method only refreshed the OAuth2 token when the token was `None` (since not self.oauth2_token) or when it is `OAuth2Token` and expired . However, if the token is any other data type other than `OAuth2Token` like dictionary  , the refresh logic was skipped.

## Why  it happened?

The refresh condition relied on this code block:

if not self.oauth2_token or (
isinstance(self.oauth2_token, OAuth2Token) and self.oauth2_token.expired
)

When `oauth2_token` was a dictionary  or any other data type other than `OAuth2Token` ,the condition evaluated to `False`. Because these data types are  truthy and not an instance of `OAuth2Token`, as result the refresh was never triggered and  the Authorization header was not added to the request.

## Why does the fix solve it?

I simplified the condition to:

if not isinstance(self.oauth2_token, OAuth2Token) or self.oauth2_token.expired


This condition is generalistic and  ensures that any token that is not an `OAuth2Token` instance (including dictionaries or other unexpected types) is treated as invalid and triggers a refresh.

After refreshing, the client obtains a valid `OAuth2Token` and correctly sets the Authorization header.

## An edge case not covered

The current tests do not cover the scenario where a dictionary token contains a valid `access_token` and a future `expires_at`. In that case, the client still forces a refresh instead of converting the dictionary into an `OAuth2Token`.
