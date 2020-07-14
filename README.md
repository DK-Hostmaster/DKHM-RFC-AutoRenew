![DK Hostmaster Logo](https://www.dk-hostmaster.dk/sites/default/files/dk-logo_0.png)

# DKHM RFC for handling of Automatic Renewal

![Markdownlint Action](https://github.com/DK-Hostmaster/DKHM-RFC-Delete-Domain/workflows/Markdownlint%20Action/badge.svg)
![Spellcheck Action](https://github.com/DK-Hostmaster/DKHM-RFC-Delete-Domain/workflows/Spellcheck%20Action/badge.svg)

## Table of Contents

<!-- MarkdownTOC bracket=round levels="1,2,3,4" indent="  " autolink="true" autoanchor="true" -->

- [Introduction](#introduction)
  - [About this Document](#about-this-document)
  - [XML and XSD Examples](#xml-and-xsd-examples)
- [Description](#description)
- [XSD Definition](#xsd-definition)
- [References](#references)

<!-- /MarkdownTOC -->

<a id="introduction"></a>
## Introduction

This is a draft and proposal for changes to the process for automatic renewal for domain names via the DK Hostmaster EPP portal/service.

<a id="about-this-document"></a>
### About this Document

We have adopted the term RFC (_Request For Comments_), due to the recognition in the term and concept, so this document is a process supporting document, aiming to serve the purpose of obtaining a common understanding of the proposed implementation and to foster discussion on the details of the implementation. The final specification will be lifted into the [DK Hostmaster EPP Service Specification](https://github.com/DK-Hostmaster/epp-service-specification) implementation and this document will be closed for comments and the document no longer be updated and it will be archived.

<a id="xml-and-xsd-examples"></a>
### XML and XSD Examples

All example XML files are available in the [DK Hostmaster EPP XSD repository](https://github.com/DK-Hostmaster/epp-xsd-files) in the [autorenew-dkhm-extension](https://github.com/DK-Hostmaster/epp-xsd-files/tree/autorenew-dkhm-extension) branch.

<a id="description"></a>
## Description

DK Hostmaster introduces the concept of auto renewal. This is will be based on a single value assigned to a designated domain name.

The default is that auto renewal is enabled, meaning that domain names will be renewed unless requested not to.

The basic request to alter this behaviour is to use the `update domain` command. The command will support both enabling and disabling auto renewal.

`info domain` will be extended with information on the current status for auto renawal.

In addition to updating the auto renewal state using by updating the setting. The setting can be set both at the time of:

- domain creation via `create domain`
- transfer via `transfer domain`

The handling of automatic renewal for transfer is described in detail in this RFC, for addional details on the DK Hostmaster implementation for the `transfer domain` command, please refer to the separate RFC: "[DKHM RFC for Transfer Domain EPP Command][DKHMRFCTRANSFER]" for details.

And last but not least the auto renewal can be disabled by using the `delete domain` command, by providing a date prior to the expiry date, auto renewal will be implicitly disables. Please see the  RFC: "[DKHM RFC for Delete Domain EPP Command][DKHMRFCDELDOM]" for details.

```xml
  <extension>
    <dkhm:autoRenew xmlns:dkhm="urn:dkhm:xml:ns:dkhm-3.2">false</dkhm:autoRenew>
  </extension>
```

The date follows the format used in the EPP protocol, which complies with [RFC:3339][RFC3339].

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

Ref: [`dkhm-3.3.xsd`](https://raw.githubusercontent.com/DK-Hostmaster/epp-xsd-files/master/dkhm-3.3.xsd)

The complete command will look as follows (example lifted from RFC:5731):

```xml
```

And the complete command with a deletion date specification (example lifted from RFC:5731 and modified):

```xml
```

<a id="xsd-definition"></a>
## XSD Definition

This XSD definition is for the proposed extension `dkhm:delDate`, which is used to communicate a deletion date differing from the default via the `delete domain` request.

```xsd
  <!-- custom: delDate  -->
  <simpleType name="delDate">
    <restriction base="dateTime" />
  </simpleType>
```

Example (lifted from above):

```xml
  <extension>
    <dkhm:delDate xmlns:dkhm="urn:dkhm:xml:ns:dkhm-3.2">2021-01-31T00:00:00.0Z</dkhm:delDate>
  </extension>
```

Ref: [`dkhm-3.3.xsd`](https://raw.githubusercontent.com/DK-Hostmaster/epp-xsd-files/master/dkhm-3.3.xsd)

:warning: The reference and file mentioned above is not released at this time, so this file might be re-versioned upon release.

<a id="references"></a>
## References

- [DK Hostmaster EPP Service Specification](https://github.com/DK-Hostmaster/epp-service-specification)
- [DK Hostmaster EPP Service XSD Repository](https://github.com/DK-Hostmaster/epp-xsd-files)
- [DKHM RFC for Delete Domain EPP Command][DKHMRFCDELDOM]
- [RFC:5730 "Extensible Provisioning Protocol (EPP)"][RFC5730]
- [RFC:5731 "Extensible Provisioning Protocol (EPP) Domain Name Mapping"][RFC5731]

[RFC5730]: https://www.rfc-editor.org/rfc/rfc5730.html
[RFC5731]: https://www.rfc-editor.org/rfc/rfc5731.html
[DKHMRFCDELDOM]: https://github.com/DK-Hostmaster/DKHM-RFC-Delete-Domain
[DKHMRFCTRANSFER]: https://github.com/DK-Hostmaster/DKHM-RFC-Delete-Domain
