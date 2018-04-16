# Resetting a User's Password

A user's password can be reset by the user in question or by an admin. To do this, you'll need the hex-based User ID, the user/admin refresh token, and the desired password.

Here's an example for resetting the password via an HTTP request. The token is supplied through an Authorization header.

```bash
curl -X PUT -H "Content-Type: application/json" -H "Authorization:<user-or-admin-refresh-token>" -d '{"user_id": <user-id-to-change-password>, "data": {"new_password": <new-password>}}' http://localhost:9080/auth/password
```

The JSON payload is:

```javascript
{
    "user_id": "5d0fbc03f4d7ab1dcc127aace224270c",
    //optionalifuserischanginghis/herownpassword
    "data":
        {
            "new_password": "new-password"
        }
}
```

The refresh token can be retrieved by calling `user.token` after logging in via `Realm.Sync.User`



Not what you were looking for? [Leave Feedback](https://realm3.typeform.com/to/A4guM3) 

