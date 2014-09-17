# SAML for Web Developers

Interested in adding SAML (Security Assertion Markup Language) support to your
app? This post explains the basic single sign-on flows for web applications. If
you have questions or feedback, please [file an
issue](https://github.com/jch/saml/issues).

## Benefits

SAML lets you manage user identities and authorizations across multiple apps.
It's similar to how we use our Google identity to login to YouTube, or our
Facebook identity for.... errr everything else.

Large companies love single sign-on. Instead of using OAuth, they use SAML. And
instead of Farmville, it's Enterprise Software™. Po-tay-toes, Po-tah-toes.

The SAML standard has been around for a long time, with the latest version (2.0)
released in 2005. It's widely adopted by large companies, and there's a whole
industry (identity access management, IAM) dedicated to supporting it.

How does it make your users' lives easier?

- Single sign-on. Logged into Windows? Sweet. That means the user's also logged into your app. No additional username or password needed.
- Automatic user creation (provisioning) based on your company identity.
- User profiles pre-filled from their company profile (via LDAP or other directory service).
- Single source of truth for managing users. Administrators can onboard and offboard users from one place. They can also manage app access for groups of users at a time, rather than individually.

## Single sign on

There are many complex use cases for SAML, but let's start with a simple example
to get familiar with the jargon.

> Alice needs to schedule a work trip. Her company has two internal web
> applications; One for booking flights, and another for booking a rental car.

In SAML speak, Alice is our **principal**, the user that we're trying to
authenticate and learn about. The two applications are the **service
providers**.

> She visits https://flights.acme-corp.biz/trips and is redirected to
> https://idp.acme-corp.biz?SAMLRequest=... to single sign-on.

When the principal, Alice on her browser, tries to access a protected resource
(/flights), the service provider (https://flights.acme-corp.biz) doesn't present
her with a login page. Instead, it redirects her to an **identity provider**
(https://idp.acme-corp.biz) with an authentication request in the query
parameter. The identity provider uses the authentication request to tell which
service provider is making the request. Alice is now logged into the identity
provider. In this context, we can also call the identity provider a **session
authority** and say that Alice has a valid session with it. This becomes
important later on when we talk about how she books a car rental.

![](/images/authn-request.png)

*Tip: I avoid abbreviations and acronyms in this guide, but you will commonly
see service provider abbreviated as SP, and identity providers abbreviated as
idP.*

> Alice types her username and password into https://idp.acme-corp.biz and is
> redirected back to https://flights.acme-corp.biz/flights.

After the identity provider authentication successfully authenticates Alice, it
sends a response back to the service provider saying authentication was
successful, and releasing **assertions** about the principal. Assertions are
statements about the principal. They typically include information like email,
full name, and other contact information. The service provider can use these to
update an existing user's profile, or to provision a new user. (Provisioning is
a fancy term for "creating a new thing").

> After booking her flight, she visits https://cars.acme-corp.biz/rentals. She
> is once again redirected to https://idp.acme-corp.biz?SAMLRequest=..., but
> she's immediately redirected back to https://cars.acme-corp.biz/rentals
> without having to type her password again.

Why didn't Alice need to re-authenticate? Remember that when she redirected to
the identity provider for booking her flight, she established a session with the
identity provider. When this other service provider (https://cars.acme-corp.biz)
sent a authentication request, the identity provider saw that Alice had
previously authenticated because of her active session. There's no need for
Alice to re-authenticate again, so the identity provider redirects her back to
the site she wanted with a successful response.

![](/images/authn-request-subsequent.png)

***Tip**: Just like how an identity provider can be called a session authority, the
complement for service providers is to be called a **session participant**.*

## Bindings

In the single sign-on example, I glossed over how a user is "redirected" between
the identity providers and service providers. In SAML, a **binding** is
describes how messages should be encoded, and the underlying transport protocol
to carry them. For web single sign-on, two common bindings are the "HTTP
Redirect Binding" and the "HTTP Post Binding". Their names hint at their
function. For example, we can specify that initial authentication requests from
a service provider to an identity provider should happen with the redirect
binding; The response from identity provider to service provider should happen
with the post binding.

This is a common configuration, but SAML also supports other bindings like SOAP,
URI, and Artifact.

## Security

SAML responses include **conditions** under which the response is valid.
Responses are typically only good for a few minutes to prevent replay attacks
(SAMLCore 2.5.1.2). They can be optionally signed and/or encrypted with a shared
secret between **relying parties** (all identity providers and service
providers) (SAMLCore 5 and 6). It's a good idea to do everything over SSL.

## Metadata

Because SAML is such a general framework, there's a ton of choices for how to
configure it. Which binding to use? Signed or unsigned assertions / responses?
Fortunately, SAML also defines a standard for communicating these configurations
between relying parties. By publishing metadata about your service, configuring
your service could be as easy as a single click to import the metadata.

## Putting it all together

Now that we know all the pieces involved, let's implement a simple single sign
on flow for the Flights service provider. Here are our requirements:

- HTTP Redirect Binding for initiating authentication requests (configured in identity provider)
- HTTP Post Binding for receiving SAML responses (configured in service provider)
- No signed assertions or signed responses

To keep the examples simple, I'm putting the minimum set of attributes in the
XML messages. There are many more attributes and elements you can optionally
specify to tweak it to your needs. [Schema
Central][schema-central] is a great reference for
what's actually required and what's optional.

First, we need to define service provider metadata to let identity providers
know how to talk to us.

```xml
<?xml version="1.0"?>
<md:EntityDescriptor xmlns:md="urn:oasis:names:tc:SAML:2.0:metadata" entityID="flights.acme-corp.biz">
  <md:SPSSODescriptor protocolSupportEnumeration="urn:oasis:names:tc:SAML:2.0:protocol" AuthnRequestsSigned="false" WantAssertionsSigned="false">
    <md:AssertionConsumerService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST" Location="https://flights.acme-corp.biz/saml/consume" isDefault="true" index="0"/>
  </md:SPSSODescriptor>
</md:EntityDescriptor>
```

The `AssertionConsumerService` element tells identity providers to use the HTTP
POST Binding to send responses to `/saml/consume`.

When Alice requests the page `/flights`, our app will redirect the user to the
identity provider and pass along the following authentication request:

```xml
<?xml version="1.0"?>
<samlp:AuthnRequest ID="_abc123" IssueInstant="2014-09-17T20:53:21" Version="2.0">
</samlp:AuthnRequest>
```

The `AuthnRequest` must have a globally unique identifier (ID). This will show
up again when we see the response from the identity provider.

Since we're using the HTTP Redirect binding, we want to Base64 encode this XML
and append it to the url as a query parameter `SAMLRequest`. So the location we
redirect Alice to will be:

```
http://idp.acme-corp.biz/sso?SAMLRequest=PD94bWwgdmVyc2lvbj0iMS4wIj8%2BCjxzYW1scDpBdXRoblJlcXVlc3QgeG1s%0AbnM6c2FtbHA9InVybjpvYXNpczpuYW1lczp0YzpTQU1MOjIuMDpwcm90b2Nv%0AbCIgeG1sbnM6c2FtbD0idXJuOm9hc2lzOm5hbWVzOnRjOlNBTUw6Mi4wOmFz%0Ac2VydGlvbiIgSUQ9Il9hYmNkZTEyMyIgSXNzdWVJbnN0YW50PSIyMDE0LTA5%0ALTE3VDIwOjUzOjIxIiBWZXJzaW9uPSIyLjAiPgo8L3NhbWxwOkF1dGhuUmVx%0AdWVzdD4%3D%0A
```

***Tip**: Be really careful about whitespace in your requests and responses. These
can wreak havoc on digital signatures and cause errors when validating
messages.*

Alice sees the login page for the identity provider, types in the correct
credentials, and establishes a session with the identity provider. The identity
provider gathers the information it knows about Alice and builds a response with
the assertions about Alice:

```xml
<?xml version="1.0"?>
<samlp:Response ID="_rst456" InResponseTo="_abc123" IssueInstant="2014-09-17T21:06:32" Version="2.0">
  <samlp:Status>
    <samlp:StatusCode Value="urn:oasis:names:tc:SAML:2.0:status:Success"/>
  </samlp:Status>
  <saml:Assertion ID="_xyz888" IssueInstant="2014-09-17T21:06:32" Version="2.0">
    <saml:AttributeStatement>
      <saml:Attribute Name="email">
        <saml:AttributeValue>alice@acme-corp.biz</saml:AttributeValue>
      </saml:Attribute>
    </saml:AttributeStatement>
  </saml:Assertion>
</samlp:Response>
```

Again, this response is Base64 encoded, but since we're specified in our
metadata to use HTTP Post binding for receiving messages, the identity provider
will POST the encoded message back to
https://flights.acme-corp.biz/saml/consume. The location was also defined in the
metadata.

***Tip**: It's common to use the HTTP Post Binding for receiving SAML responses
because browsers have url length limits that don't work with with large
responses.*

The Flights app sees that the authentication was successful from the
`StatusCode` element, and can also learn more about the principal from the
`Assertion` element; Here we see that Alice's email is `alice@acme-corp.biz`.
Using this information, Flights can now establish a session for Alice.

## Further reading

To keep the introduction gentle and focused for web developers, I intentionally
left out a lot of cool details. What if you have native applications that
doesn't go through a web browser? What about other bindings? What if you want to
extend the spec?

The documentation is fantastic, but I recommend reading them in this order:

- [Technical Overview](https://www.oasis-open.org/committees/download.php/27819/sstc-saml-tech-overview-2.0-cd-02.pdf): Read this first.
- [Assertions and Protocols](http://docs.oasis-open.org/security/saml/v2.0/saml-core-2.0-os.pdf): The bulk of the specification.

These are interesting to skim, but work better as reference docs rather than a
guide:

- [Metadata](http://docs.oasis-open.org/security/saml/v2.0/saml-metadata-2.0-os.pdf)
- [Bindings](http://docs.oasis-open.org/security/saml/v2.0/saml-bindings-2.0-os.pdf)

## Resources

- [Firefox SAML Tracer plugin](https://addons.mozilla.org/en-US/firefox/addon/saml-tracer/): decode and view SAML messages
- [SAML Developer Tools](https://www.samltool.com)
- [Schema Central][schema-central]: XML schema reference for SAML messages


[schema-central]: http://www.datypic.com/sc/saml2/ss.html "XML schema reference for SAML messages"
