![DK Hostmaster Logo](https://www.dk-hostmaster.dk/sites/default/files/dk-logo_0.png)

# DKHM RFC for handling of Automatic Renewal or Expiration

![Markdownlint Action](https://github.com/DK-Hostmaster/DKHM-RFC-AutoRenew/workflows/Markdownlint%20Action/badge.svg)
![Spellcheck Action](https://github.com/DK-Hostmaster/DKHM-RFC-AutoRenew/workflows/Spellcheck%20Action/badge.svg)

2020-08-25
Revision: 1.0

## Table of Contents

<!-- MarkdownTOC bracket=round levels="1,2,3,4" indent="  " autolink="true" autoanchor="true" -->

- [Introduction](#introduction)
  - [About this Document](#about-this-document)
  - [License](#license)
  - [XML and XSD Examples](#xml-and-xsd-examples)
- [Description](#description)
- [XSD Definition](#xsd-definition)
- [References](#references)

<!-- /MarkdownTOC -->

<a id="introduction"></a>
## Introduction

This is a draft and proposal for changes to the process for automatic renewal for domain names via the DK Hostmaster EPP portal/service. The specification briefly touches on the registrar portal service, which mimicks the EPP service for consistency.

The overall [description of the concept][CONCEPT] of the registrar model offered by DK Hostmaster A/S provided as a general overview, where this RFC digs into the details of automatic renewal in the context of an implementation proposal.

<a id="about-this-document"></a>
### About this Document

We have adopted the term RFC (_Request For Comments_), due to the recognition in the term and concept, so this document is a process supporting document, aiming to serve the purpose of obtaining a common understanding of the proposed implementation and to foster discussion on the details of the implementation. The final specification will be lifted into the [DK Hostmaster EPP Service Specification][DKHMEPPSPEC] implementation and this document will be closed for comments and the document no longer be updated and it will be archived.

<a id="license"></a>
### License

This document is copyright by DK Hostmaster A/S and is licensed under the MIT License, please see the separate LICENSE file for details.

<a id="xml-and-xsd-examples"></a>
### XML and XSD Examples

All example XML files are available in the [DK Hostmaster EPP XSD repository][DKHMXSDSPEC].

The proposed extensions and XSD definitions are available in the  [3.2 candidate][DKHMXSD3.2] of the DK Hostmaster XSD, which is currently a draft and work in progress and marked as a  _pre-release_.

<a id="description"></a>
## Description

DK Hostmaster introduces the concept of auto renewal. This is will be based on a single value assigned to a designated domain name.

The default is that auto renewal is enabled, meaning that domain names will be renewed unless requested not to.

The basic request to alter this behaviour is to use the `update domain` command. The command will support both enabling and disabling auto renewal.

`info domain` will be extended with information on the current status for auto renewal.

In addition to updating the auto renewal state using by updating the setting. The setting can be set both at the time of:

- domain creation via `create domain`
- transfer via `transfer domain`

The handling of automatic renewal for transfer is described in detail in this RFC, for additional details on the DK Hostmaster implementation for the `transfer domain` command, please refer to the separate RFC: "[DKHM RFC for Transfer Domain EPP Command][DKHMRFCTRANSFER]" for details.

And last but not least the auto renewal can be disabled by using the `delete domain` command, by providing a date prior to the expiry date, auto renewal will be implicitly disables. Please see the  RFC: "[DKHM RFC for Delete Domain EPP Command][DKHMRFCDELDOM]" for details.

```xml
  <extension>
    <dkhm:autoRenew xmlns:dkhm="urn:dkhm:xml:ns:dkhm-3.2">false</dkhm:autoRenew>
  </extension>
```

The XSD for the extension look as follows:

```xsd
  <!-- custom: autoRenew  -->
  <simpleType name="autoRenew">
    <restriction base="token">
        <enumeration value="true"/>
        <enumeration value="false"/>
    <restriction/>
  </simpleType>
```

Ref: [`dkhm-3.2.xsd`][DKHMXSD3.2]

The complete command for disabling automatic renewal will look as follows (example lifted from RFC:5731 and modified):

```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<epp xmlns="urn:ietf:params:xml:ns:epp-1.0">
  <command>
    <update>
      <domain:update
       xmlns:domain="urn:ietf:params:xml:ns:domain-1.0">
        <domain:name>eksempel.dk</domain:name>
        <domain:chg>
          <domain:registrant>DKHM1-DK</domain:registrant>
        </domain:chg>
      </domain:update>
    </update>
    <extension>
      <dkhm:autoRenew xmlns:dkhm="urn:dkhm:xml:ns:dkhm-3.2">false</dkhm:autoRenew>
    </extension>
    <clTRID>ABC-12345</clTRID>
  </command>
</epp>
```

And the complete command for enabling automatic renewal will look as follows (example lifted from RFC:5731 and modified):

```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<epp xmlns="urn:ietf:params:xml:ns:epp-1.0">
  <command>
    <update>
      <domain:update
       xmlns:domain="urn:ietf:params:xml:ns:domain-1.0">
        <domain:name>eksempel.dk</domain:name>
        <domain:chg>
          <domain:registrant>DKHM1-DK</domain:registrant>
        </domain:chg>
      </domain:update>
    </update>
    <extension>
      <dkhm:autoRenew xmlns:dkhm="urn:dkhm:xml:ns:dkhm-3.2">true</dkhm:autoRenew>
    </extension>
    <clTRID>ABC-12345</clTRID>
  </command>
</epp>
```

The alteration of the value will be reflected in the `info domain` response using the same extension (example lifted from DK Hostmaster EPP service specification revision 3.8 and modified):

```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>

<epp xmlns="urn:ietf:params:xml:ns:epp-1.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="urn:ietf:params:xml:ns:epp-1.0 epp-1.0.xsd">
  <response>
    <result code="1000">
      <msg>Info result</msg>
    </result>
    <resData>
      <domain:infData xmlns:domain="urn:ietf:params:xml:ns:domain-1.0">
        <domain:name>dk-hostmaster.dk</domain:name>
        <domain:roid>DK_HOSTMASTER_DK-DK</domain:roid>
        <domain:status s="serverUpdateProhibited"/>
        <domain:status s="serverTransferProhibited"/>
        <domain:status s="serverDeleteProhibited"/>
        <domain:registrant>DKHM1-DK</domain:registrant>
        <domain:ns>
          <domain:hostObj>auth01.ns.dk-hostmaster.dk</domain:hostObj>
          <domain:hostObj>auth02.ns.dk-hostmaster.dk</domain:hostObj>
          <domain:hostObj>p.nic.dk</domain:hostObj>
        </domain:ns>
        <domain:host>auth01.ns.dk-hostmaster.dk</domain:host>
        <domain:host>auth02.ns.dk-hostmaster.dk</domain:host>
        <domain:host>venteliste1.dk-hostmaster.dk</domain:host>
        <domain:host>venteliste2.dk-hostmaster.dk</domain:host>
        <domain:host>blocked1.ns.dk-hostmaster.dk</domain:host>
        <domain:host>blocked2.ns.dk-hostmaster.dk</domain:host>
        <domain:clID>DKHM1-DK</domain:clID>
        <domain:crID>DKHM1-DK</domain:crID>
        <domain:crDate>1998-01-19T00:00:00.0Z</domain:crDate>
        <domain:exDate>2022-03-31T00:00:00.0Z</domain:exDate>
      </domain:infData>
    </resData>
    <extension>
        <dkhm:autoRenew xmlns:dkhm="urn:dkhm:xml:ns:dkhm-3.2">true</dkhm:autoRenew>
    </extension>
    <trID>
      <clTRID>fee9352765ab62fefc69558d3f4e0eed</clTRID>
      <svTRID>CF056EB6-D2CA-11E9-8DAB-C853983F0358</svTRID>
    </trID>
  </response>
</epp>
```

As described the flag can be set at the time of transfer or creation.

A creation command would look as follows:

```xml
<?xml version="1.0" encoding="utf-8"?>
<epp xmlns="urn:ietf:params:xml:ns:epp-1.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="urn:ietf:params:xml:ns:epp-1.0 epp-1.0.xsd">
    <command>
        <create>
            <domain:create xmlns:domain="urn:ietf:params:xml:ns:domain-1.0" xsi:schemaLocation="urn:ietf:params:xml:ns:domain-1.0 domain-1.0.xsd">
                <domain:name>eksempel.dk</domain:name>
                <domain:period unit="y">1</domain:period>
                <domain:ns>
                    <domain:hostObj>ns1.dk-hostmaster.dk</domain:hostObj>
                    <domain:hostObj>ns2.dk-hostmaster.dk</domain:hostObj>
                </domain:ns>
                <domain:registrant>DKHM1-DK</domain:registrant>
                <domain:authInfo>
                    <domain:pw />
                </domain:authInfo>
            </domain:create>
        </create>
        <extension>
            <dkhm:orderconfirmationToken xmlns:dkhm="urn:dkhm:params:xml:ns:dkhm-3.2">testtoken</dkhm:orderconfirmationToken>
            <dkhm:autoRenew xmlns:dkhm="urn:dkhm:xml:ns:dkhm-3.2">true</dkhm:autoRenew>
        </extension>
        <clTRID>92724843f12a3e958588679551aa988d</clTRID>
    </command>
</epp>
```

Do note the above example also includes the `dkhm:orderconfirmationToken`, which is not related to `dkhm:autoRenew`, but can co-exist as demonstrated by the example.

For transfer an example using the same extension would look as follows (example lifted from RFC:5731 and modified):

```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<epp xmlns="urn:ietf:params:xml:ns:epp-1.0">
  <command>
    <transfer op="request">
      <domain:transfer
       xmlns:domain="urn:ietf:params:xml:ns:domain-1.0">
        <domain:name>eksempel.dk</domain:name>
        <domain:authInfo>
          <domain:pw>DKHM1-DK-098f6bcd4621d373cade4e832627b4f6</domain:pw>
        </domain:authInfo>
      </domain:transfer>
    </transfer>
    <extension>
        <dkhm:autoRenew xmlns:dkhm="urn:dkhm:xml:ns:dkhm-3.2">true</dkhm:autoRenew>
    </extension>
    <clTRID>ABC-12345</clTRID>
  </command>
</epp>
```

Since the support for the EPP command `transfer domain` is only a proposal, the above example relies on the outcome of the RFC. please refer to the separate RFC: "[DKHM RFC for Transfer Domain EPP Command][DKHMRFCTRANSFER]" for more details.

In order to minimise the use of extensions, the registrar will be allowed to define default values for autorenewal behaviour and these defaults will be used by the above commands, where the extensions are not used.

If a registrar specifies the default behaviour of own account to be that auto renewal should not happen and this setting is now overwritten by using the extension for the specific commands, auto renewal will be set to `false`.

| Command\Default | Auto Renew | Expire  |
|-----------------|------------|---------|
| update          |    N/A     |   N/A   |
| create          |   `true`   | `false` |
| transfer        |   `true`   | `false` |

The `update domain` will only rely on the use of the extension for explicitly setting the auto renewal for the designated domain name. Whereas `create domain` and `transfer` domain will use the default setting, unless explicitly overwritten using the extension and of course specifying a value holding the opposite value of the default. Specifying the same value as the default using the extension will of course not have any effect.

Administration of these values for controlling the default behaviour will have to be done via the Registrar Portal, since account level commands are not offered by the EPP service.

<a id="xsd-definition"></a>
## XSD Definition

This XSD definition is for the proposed extension `dkhm:autoRenew`, which is used to communicate a the setting of auto renewal for a designated domain name.

```xsd
  <!-- custom: autoRenew  -->
  <simpleType name="autoRenew">
    <restriction base="token">
        <enumeration value="true"/>
        <enumeration value="false"/>
    <restriction/>
  </simpleType>
```

Example (lifted from above):

```xml
  <extension>
    <dkhm:autoRenew xmlns:dkhm="urn:dkhm:xml:ns:dkhm-3.2">false</dkhm:autoRenew>
  </extension>
```

Ref: [`dkhm-3.2.xsd`][DKHMXSD3.2]

:warning: The reference and file mentioned above is not released at this time, so this file might be re-versioned upon release.

<a id="references"></a>
## References

- [DK Hostmaster EPP Service Specification][DKHMEPPSPEC]
- [DK Hostmaster EPP Service XSD Repository][DKHMXSDSPEC]
- [DKHM RFC for Delete Domain EPP Command][DKHMRFCDELDOM]
- [RFC:5730 "Extensible Provisioning Protocol (EPP)"][RFC5730]
- [RFC:5731 "Extensible Provisioning Protocol (EPP) Domain Name Mapping"][RFC5731]

[RFC5730]: https://www.rfc-editor.org/rfc/rfc5730.html
[RFC5731]: https://www.rfc-editor.org/rfc/rfc5731.html
[DKHMRFCDELDOM]: https://github.com/DK-Hostmaster/DKHM-RFC-Delete-Domain
[DKHM1-DK-098f6bcd4621d373cade4e832627b4f6]: https://github.com/DK-Hostmaster/DKHM-RFC-Delete-Domain
[DKHMXSD3.2]: https://github.com/DK-Hostmaster/epp-xsd-files/blob/master/dkhm-3.2.xsd
[CONCEPT]: https://www.dk-hostmaster.dk/en/new-basis-collaboration-between-registrars-and-dk-hostmaster
[DKHMEPPSPEC]: https://github.com/DK-Hostmaster/epp-service-specification
[DKHMXSDSPEC]: https://github.com/DK-Hostmaster/epp-xsd-files
