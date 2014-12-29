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

### Configure a connection to a LDAP backend

https://wiki.shibboleth.net/confluence/display/SHIB2/ResolverLDAPDataConnector

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
