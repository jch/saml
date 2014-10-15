# Single logout

The complement to single sign on is single logout, frequently abbreviated as
SLO. It allows users to logout of their active session from multiple
applications from one action. There are two types of SLO service provider
initiated, and identity provider initiated.

## Service provider initiated single logout

Logging out of any application with single signed on sessions will send a logout
request to the identity provider, who will terminiate the session and forward
the logout request to all other service providers.

![]() TODO: SP initiated SLO

## Identity provider initiated single logout

Some SAML installations will have a web frontend for their identity provider to
list all the applications users have. Identity provider initiated logout is when
a user decides to logout of all their active applications from the identity
provider rather than from a specific application. Service provider initiated
single logout is required by the SAML 2.0 specification, but identity provider
initiated logout is optional.

![]() TODO: idP initiated SLO

## Basic example

Revisiting our single sign on example, Alice used single sign on to login to
both service providers "Flights" and "Cars" via an identity provider at
https://idp.acme-corp.biz. Suppose she clicks the logout link on the "Flights"
application; This causes "Flights" to send the identity provider a
`LogoutRequest`.

```xml
<?xml version="1.0"?>
<samlp:LogoutRequest Destination="https://idp.acme.biz">
  <saml:NameID>alice@acme.biz</saml:NameID>
</samlp:LogoutRequest>
```

<em>Tip: Like all requests in SAML, `LogoutRequest` should be signed so the
recipient knows the message is valid. This example is unsigned to keep the XML
readable.</em>

The `LogoutRequest` specifies who is being logged out with the `NameID` element.
Our principal is Alice, and we reference her by her email. In other systems,
this might be a login name or some other unique identifier.

Once the identity provider receives the `LogoutRequest`, it will create a
`LogoutRequest` to send to other session participants, namely, the "Cars"
application.

```xml
<?xml version="1.0"?>
<samlp:LogoutRequest Destination="https://cars.acme.biz">
  <saml:NameID>alice@acme.biz</saml:NameID>
</samlp:LogoutRequest>
```

TODO: cars will terminate session, send a LogoutResponse.

Finally it'll terminate Alice's session and respond with a `LogoutResponse` message:

```xml
<?xml version="1.0"?>
<samlp:LogoutResponse>

</samlp:LogoutResponse>
```

## Challenges

We've only explored the happy path so far. Unfortunately, there are many
challenges to single logout from a user experience perspective and from a
technical perspective.

- What happens if a user thinks they're logged out when they close their window?
- What happens if an application receives a `LogoutRequest` while in the middle of an important transaction?
- How do we logout of applications that only track sessions with domain cookies?

TODO: explain SLO challenges

## Further reading

- [Assertions and Protocols: 3.7 Single Logout Protocol](http://docs.oasis-open.org/security/saml/v2.0/saml-core-2.0-os.pdf)
- [Single Logout](http://www.slideshare.net/erlang/single-logout)
