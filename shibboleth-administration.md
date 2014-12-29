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
- Test attribute configuration from the command line

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

https://wiki.shibboleth.net/confluence/display/SHIB2/ResolverSAML2NameIDAttributeDefinition
https://wiki.shibboleth.net/confluence/display/SHIB2/ResolverSimpleAttributeDefinition
https://wiki.shibboleth.net/confluence/display/SHIB2/ResolverMappedAttributeDefinition

https://wiki.shibboleth.net/confluence/display/SHIB2/IdPAddAttribute#IdPAddAttribute-AttributeEncoding

### Release attributes to SP

https://wiki.shibboleth.net/confluence/display/SHIB2/IdPAddAttributeFilter

## Test attribute configuration from the command line

https://wiki.shibboleth.net/confluence/display/SHIB2/AACLI
