# Password Reset and Email Confirmation

It is possible for users of the [Username/Password](https://docs.realm.io/platform/~/edit/drafts/-LAsnHmaA1RnER9TsPEx/v/3.x/self-hosted/customize/authentication/username-password) authentication provider to ask for email confirmation or a password reset. To do this, the clients will make an API call to the Realm Object Server in order to generate a token that will be used to complete the flow \(password reset or email confirmation\), this token is sent by email from the Realm Object Server. 

## Configuring Realm Object Server to send emails

Sending a confirmation email or password reset email only applies to the [Password Provider](https://docs.realm.io/platform/~/edit/drafts/-LAsnHmaA1RnER9TsPEx/v/3.x/self-hosted/customize/authentication/username-password), we need to configure this provider so it can send emails by adding an `emailHandlerConfig` option.

#### emailHandlerConfig

It's an object that contains the configuration options for the built-in email handler. It allows  you to configure some basic properties, such as sender email, smtp, or an email template.

{% hint style="info" %}
We're going to edit the `my-app/src/index.ts` configuration of our project \([generated previously](https://docs.realm.io/platform/self-hosted/running-the-server) by `ros init my-app`\)
{% endhint %}

### SMTP parameters

```typescript
authProviders: [new auth.PasswordAuthProvider({
    emailHandlerConfig: {
        baseUrl: 'http://localhost:9080' // Used to replace %BASE_URL% in email templates
        connectionString: 'smtp://smtp_username:smtp_password@smtp.example.com',
        from: 'foo@bar.com'
    }
})],
```

{% hint style="info" %}
The built-in email handler uses [nodemailer](https://nodemailer.com) to send out emails.
{% endhint %}

#### connectionString

The [connection url](https://nodemailer.com/smtp/) to configure the nodemailer transporter.

#### from

The email address of the sender. All email addresses can be plain `sender@server.com` or formatted `"Sender Name" sender@server.com`.

### Customizing the email templates 

Realm Object Server provides a default email template for both password reset and email confirmation operations, if you wish to customize one or both of them, you can do so by setting the following  optionals variables: `resetActionConfig` and/or `confirmActionConfig,` both take the same arguments:

#### subject

The subject of the email that will be sent.

#### textTemplate

The template for the text version of the email. It may contain placeholders, such as `%TOKEN%` or `%IP%`.

#### htmlTemplate

The template for the html version of the email. It may contain placeholders, such as `%TOKEN%` or `%IP%`

{% hint style="info" %}
`%TOKEN%` `%BASE_URL%`and `%IP%` are placeholders that will be replaced by the actual values before sending the email by the Realm Object Server. 
{% endhint %}

  
Here's an example that customizes the password reset email, but keeps the default for email confirmation:

```typescript
authProviders: [new auth.PasswordAuthProvider({
     emailHandlerConfig: {
        connectionString: 'smtp://smtp_username:smtp_password@smtp.example.com',
        from: 'foo@bar.com',
        resetActionConfig: {
            subject: 'Forgot your password?',
            textTemplate: `Please use this link below to reset your password and access your account.
                          %BASE_URL%/reset-password?token=%TOKEN%`,
            htmlTemplate: `<html>
            <body>
                 <h3>Did you forget your password?</h3>
                 <a href="%BASE_URL%/reset-password?token=%TOKEN%">Reset your password</a>
            </body>
          </html>`
        },
      } 
    })],
```

### Deep linking 

By default Realm Object Server will send an email that will redirect to `https://ros-url/some-page?token=%TOKEN%` where a generic UI will be shown to complete the password reset. However, it's possible to customize this URL,  e.g. change it to `myapp://update-account/%TOKEN%` to allow the client application to handle it itself \(using [deep linking](https://en.wikipedia.org/wiki/Mobile_deep_linking)\) . It's the developer's responsibility to extract the token, then complete the password reset or email confirmation flow within the application. This can be achieved by overriding the default value of `baseUrl`.

#### baseUrl _\(optional\)_:

Define the `%BASE_URL%` that will be used inside the email template.   
Example, with the  following configuration :

```typescript
emailHandlerConfig: {
        connectionString: 'smtp://smtp_username:smtp_password@smtp.example.com',
        from: 'foo@bar.com',
        baseUrl: 'myapp://update-account/'
}
```

You'll get these links generated inside the emails:

* Reset password

```text
myapp://update-account/reset-password?token=005f77885fd1f45bf69fb72d7a82374d53609e5182d6a9b07355fa033cdd67d1
```

* Email confirmation

```text
myapp://update-account/confirm-email?token=b21bf2cda3e5245cfc27235248718e1293e16070aef962ff02629ee2cde3e30c
```



