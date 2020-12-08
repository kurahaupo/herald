---
layout: docs
title: Payload Extended Data Specification
description: Payload Extended Data Specification
menubar: docs_menu
---

# Payload Extended Data Specification

This page formally specifies the Extended Data Payload. This is referenced in both the
[Secured Payload Specification]({{"/specs/payload-secured" | relative_url }}) and the
[Beacon Payload Specification]({{"/payload/payload-beacon" | relative_url }}).

This specification is a **DRAFT**

## Changes and errata

The below is the version history:-

|Date|Author|Change summary|
|---|---|---|
|2020-12-05|[Adam Fowler](https://github.com/adamfowleruk/)|Initial draft content|

## Contents

**NOTE:** No contents or formal notation as of yet. Placeholder content until formal spec format decided.

## Figures

None

## Packet data format

Note all values are Big Endian (Network Order) and optional by default. 

@startmermaid
graph LR
  A(Data Code<br>1 byte<br>0x00-0x3F<br>Reserved<br>_) --> B(Data Length<br> <br>1 byte<br>uint8<br>_)
  B-->C(Data<br><br>1+ bytes<br>binary<br>_)
@endmermaid

Note there may be multiple sections. Below is an example showing three extended data sections:-

@startmermaid
graph LR
  A(Data Code<br>1 byte<br>0x01<br>RSSI desc<br>_) --> B(Data Length<br>1 byte<br>1<br> <br>_)
  B --> C(  Data  <br>1 byte <br>0x00<br>raw<br>_)
  C -.-> D(Data Code<br>1 byte<br>0x02<br>RSSI error<br>_) --> E(Data Length<br>1 byte<br>1<br> <br>_)
  E --> F(  Data  <br>1 byte <br>2<br> <br>_)
  F -.-> G(Data Code<br>1 byte<br>0x05<br>Approx Geo<br>_) --> H(Data Length<br>1 byte<br>3<br> <br>_)
  H --> I(  Data  <br>4 bytes<br>S41<br> <br>_)
@endmermaid


## Data values supported

Any data item with no value should not be included in the data payload. 
I.e. 0 byte length extended segments are omitted.

|Code|Description|Format and size|Example|
|---|---|---|---|
|0x00|Extension data area format version ID (assume “1” if not present).|uint8|
|0x01|RSSI description.|1 byte hex| 0x00 = raw<br>0x01 = sample mean<br>0x02 = running mean<br>0x03 = sample mode<br>0x04 = running mode|
|0x02|RSSI error bound - Expressed as +/- RSSI.|uint8|3|
|0x03|Sonar range estimate - decimal metres (size = type of float, use C++ standard types).|Float. E.g. float-32 or float-64|6.54|
|0x04|Sonar error bound - Expressed as +/- metres, single value. 4+ bytes, but float. E.g. float-32 or float-64. 0x03 must be present.|Same as 0x03|1.08|
|0x05|Approximate geo location (textual) - E.g. in UK “Postal District” which is ~6000 homes.|1+ bytes UTF-8 |'S41' or '53800'|
|0x06|Approximate geo co-ordinates - EPSG:4826 (WGS 84) - minutes (0.0166 degrees), accurate to ~1852 metres|Two signed int16. Lat then Lon|
|0x07|As 0x06, but EPSG:3857 (Web mercator projection)|As 0x06||
|0x08|As 0x06, but EPSG:7789 (ITRF2014)|As 0x06||
|0x09<br>-<br>0x0F|Reserved for future geo data sections|1+ bytes||
|0x10|Textual short name description of location|1+ bytes UTF-8 String|Joe's Pizza|
|0x11|Textual short name description of beacon location within premises|1+ bytes UTF-8 String|Seating Area B|
|0x12|Textual disambiguation (optional) for beacon area. Useful for large rooms such as sports halls, football stadiums, shopping centres/malls.|1+byte UTF-8 String|East Wall or West Wall|
|0x13|Location information URL (optional). MUST be https.|1+ byte UTF-8 URL|https://www.someurl.com/myvenue|
|0x3E|Indicates the following data is encrypted for decryption by the originating phone|1+ bytes, binary|
|0x3F|Indicates the following data is encrypted for decryption by the originating phone’s health service/authority|1+ bytes, binary|
|0x40<br>-<br>0x7F|Reserved for future Herald standard use|
|0x80<br>-<br>0xFF|Custom country / application use|
