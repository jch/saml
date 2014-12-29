# Shibboleth Administration

This is an introduction to installing and configuring Shibboleth identity
provider (idP) for single sign-on with a service provider (SP). After reading
this guide, you will be able to:

- Install Shibboleth idP
- Define a new SP
- Add attributes
  - Configure a connection to a LDAP backend
  - Define attributes based on LDAP directory
  - Release attributes to SP
- Troubleshoot attribute definitions

## Install Shibboleth idP

https://wiki.shibboleth.net/confluence/display/SHIB2/IdPInstall

```sh
wget --no-verbose http://shibboleth.net/downloads/identity-provider/2.4.2/shibboleth-identityprovider-2.4.2-bin.tar.gz
tar xf shibboleth-identityprovider-2.4.2-bin.tar.gz
cd shibboleth-identityprovider-2.4.2
sudo ./install.sh
```

## Define a new SP

https://wiki.shibboleth.net/confluence/display/SHIB2/IdPMetadataProvider

## Add attributes

https://wiki.shibboleth.net/confluence/display/SHIB2/IdPAddAttribute

In SAML, the user being single signed-on is called the **principal**.
Information about the principal is defined via Shibboleth idP **attributes**.
For example, common attributes may include full names, emails, departments,
phone numbers. An attribute is defined by the following:

- **Data connector**: Connectors define where the raw information about principals are retrieved from. e.g. relational database, LDAP.
- **Attribute definition**: Definitions give an attribute an ID for reference, and define optional data transformations.
- **Attribute encoding**: Within a definition, this specifies how the value of an attribute should be encoded.
- **Attribute filter policy**: This defines who (SP's) can access attributes, and under what conditions.

### Configure a connection to a LDAP backend

https://wiki.shibboleth.net/confluence/display/SHIB2/ResolverLDAPDataConnector

```xml
<!-- conf/attribute-resolver.xml -->
<resolver:DataConnector xsi:type="LDAPDirectory" xmlns="urn:mace:shibboleth:2.0:resolver:dc"
  id="myLDAP"
  ldapURL="LDAP_URL"
  baseDN="BASE_DN"
  principal="LDAP_ADMIN_DN"
  principalCredential="LDAP_ADMIN_PASSWORD"
  lowercaseAttributeNames="true">
</resolver:DataConnector>
```

### Define attributes based on LDAP directory

https://wiki.shibboleth.net/confluence/display/SHIB2/IdPAddAttribute#IdPAddAttribute-AttributeDefinition

There are many different `AttributeDefinition` types; Below is an example of
three common definitions.

```xml
<!--
  Simple attribute definition:
    https://wiki.shibboleth.net/confluence/display/SHIB2/ResolverSimpleAttributeDefinition

  This defines an attribute, `emails`, with no transformation from the data
  connector (myLDAP). Values come from the LDAP attribute `mail`.

  `id` - provides a way to reference this definition
  `sourceAttributeID` - the name of the source attribute from resolver:Dependency. e.g. `mail` is the attribute returned by `myLDAP`.
  `xsi:type="enc:SAML2String"` - encode this attribute as a string. Other encoders at https://wiki.shibboleth.net/confluence/display/SHIB2/IdPAddAttribute#IdPAddAttribute-AttributeEncoding
  `name` - names the attribute to be released

  Note that if the `sourceAttributeID` isn't defined, it's assumed to be the
  same as the `id`. In this case, our LDAP instance's attribute name is
  `mail`, but we want to name our shibboleth attribute `email`, so we define
  both.
-->
<resolver:AttributeDefinition xsi:type="ad:Simple" id="emails" sourceAttributeID="mail">
  <resolver:Dependency ref="myLDAP" />
  <resolver:AttributeEncoder xsi:type="enc:SAML2String" name="emails" />
</resolver:AttributeDefinition>

<!--
  NameID attribute definition:
    https://wiki.shibboleth.net/confluence/display/SHIB2/ResolverSAML2NameIDAttributeDefinition
    https://wiki.shibboleth.net/confluence/display/SHIB2/SAML2StringNameIDEncoder

  This is similar to the simple attribute definition. The difference is the
  value is encoded as a NameID, rather than a normal string.

  `xsi:type="enc:SAML2StringNameID"` - encode this attribute value as a NameID
-->
<resolver:AttributeDefinition xsi:type="ad:Simple" id="NameID" sourceAttributeID="uid">
  <resolver:Dependency ref="myLDAP" />
  <resolver:AttributeEncoder xsi:type="enc:SAML2StringNameID" nameFormat="urn:oasis:names:tc:SAML:1.1:nameid-format:unspecified" />
</resolver:AttributeDefinition>

<!--
  Mapped attribute definition:
    https://wiki.shibboleth.net/confluence/display/SHIB2/ResolverMappedAttributeDefinition

  This attribute definition does more work by deriving the value to be released
  based on the value retrieved from the data connector. Values retrieved from
  `myLDAP`'s `cn` are matched against the `SourceValue` regex, `admin.*`. If
  matched, the `ReturnValue`, `true`, becomes the attribute value; Otherwise,
  the `DefaultValue` of `false` is returned.

  Note that the order of the elements matter. `resolver:Dependency` must come
  before the `ValueMap`.
-->
<resolver:AttributeDefinition xsi:type="ad:Mapped" xmlns="urn:mace:shibboleth:2.0:resolver:ad" id="administrator" sourceAttributeID="cn">
  <resolver:Dependency ref="myLDAP" />
  <resolver:AttributeEncoder xsi:type="SAML2String" xmlns="urn:mace:shibboleth:2.0:attribute:encoder" name="administrator" />
  <DefaultValue>false</DefaultValue>
  <ValueMap>
    <ReturnValue>true</ReturnValue>
    <SourceValue>admin.*</SourceValue>
  </ValueMap>
</resolver:AttributeDefinition>
```

### Release attributes to SP

https://wiki.shibboleth.net/confluence/display/SHIB2/IdPAddAttribute#IdPAddAttribute-3.ReleasetheAttribute

```xml
<!-- Release attributes to anyone -->
<afp:AttributeFilterPolicy id="releaseEverythingToAnyone">
  <afp:PolicyRequirementRule xsi:type="basic:ANY"/>
  <afp:AttributeRule attributeID="NameID">
    <afp:PermitValueRule xsi:type="basic:ANY"/>
  </afp:AttributeRule>
  <afp:AttributeRule attributeID="emails">
    <afp:PermitValueRule xsi:type="basic:ANY" />
  </afp:AttributeRule>
  <afp:AttributeRule attributeID="administrator">
    <afp:PermitValueRule xsi:type="basic:ANY" />
  </afp:AttributeRule>
</afp:AttributeFilterPolicy>
```

## Troubleshoot attribute definitions

Use the [Attribute Authority, Command Line Interface
(AACLI)](https://wiki.shibboleth.net/confluence/display/SHIB2/AACLI) to test
whether attributes are properly configured.

```sh
$ bin/aacli.sh --configDir ./conf --principal admin1

<?xml version="1.0" encoding="UTF-8"?><saml2:AttributeStatement xmlns:saml2="urn:oasis:names:tc:SAML:2.0:assertion">
  <saml2:Attribute FriendlyName="administrator" Name="administrator" NameFormat="urn:oasis:names:tc:SAML:2.0:attrname-format:uri">
    <saml2:AttributeValue xmlns:xs="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:type="xs:string">true</saml2:AttributeValue>
  </saml2:Attribute>
  <saml2:Attribute FriendlyName="emails" Name="emails" NameFormat="urn:oasis:names:tc:SAML:2.0:attrname-format:uri">
    <saml2:AttributeValue xmlns:xs="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:type="xs:string">admin1@openldap.ghe.local</saml2:AttributeValue>
  </saml2:Attribute>
</saml2:AttributeStatement>
```

Look at the [idP
logs](https://wiki.shibboleth.net/confluence/display/SHIB2/IdPLogging) for
syntax errors, connection errors to data connectors, whether attributes are
being filtered out by attribute filter policies. `idp-process.log` is the most
useful.
