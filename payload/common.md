---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: docs
title: Herald common contact tracing header
description: Common header data for contact tracing interoperability
menubar: docs_menu
---

# Common Herald Contact Tracing data

Technically this header is part of the inner payload, but the data is common to both
the [Herald simple inner payload]({{"/payload/simple" | relative_url }}) or
the [Herald secured payload]({{"/payload/secured" | relative_url }}) and can even be used with your own
contact tracing specific custom [Inner payload]({{"/payload/inner" | relative_url }})

## What does the common header provide?

The below information is appended to the [Herald Envelope payload]({{"/payload/envelope" | relative_url }}):-

Note: All numbers are Big Endian (network order).

- Read ID payload
  - Client ID - 16 byte Ephemeral rotating identifier (generated via a callback in the Herald protocol). Could use TOTP to be generated. (not required)
  - Routing code - 4 byte freeform area, specific to the protocol identifier, for routing data or specifying legal jurisdiction/company ownership. For contact tracing this is used as follows:-
    - Country code - 2 byte unsigned integer, ISO-3166-1 NUMERIC code for a country (E.g. 826 is the UK)
    - State code - 2 byte unsigned integer, defined by the country in country code, to identify the part of the country or a commercial or other entity within it
  - My TX power - 2 byte float, current tx power of the device whose ID is being read
  - Inner payload data - remaining bytes are reserved for the inner payload. MUST be either fixed length, or you can provide a parser that detects the end of this payload (See calling card / read characteristic, below)
- Calling card / ID write payload
  - Same content as for Read ID payload except after TX power and before inner payload you also have the following (included in the signature)
  - Your RSSI - 2 byte signed integer. RSSI seen of the written to phone by the writer. Needed to support the (14% in the UK) phones that do not support advertising and so cannot be read from
- Calling card / nearby devices read payload - most recent devices only
  - Multiple payloads, consisting of:-
    - Payload type - 1 byte integer. 0 = direct read, 1 = direct write, other values reserved for future use
    - Received time - 4 byte signed integer, unix epoch seconds when the echange was observed by the device
    - Payload data - as per individual read/write payloads, above. You MUST either configure a fixed length value in the Herald outer payload, or provide your own custom parser to split each shared device payload.
- Calling card / nearby devices write payload (disabled by default in the Herald protocol)
  - Exactly the same as for calling card read payload - most recent devices only

## How do I specify an inner payload?

Implement the callbacks to provide a [custom inner payload]({{"/payload/inner" | relative_url }}) data or use
the [Herald simple inner payload]({{"/payload/simple" | relative_url }}) or the [Herald secured payload]({{"/payload/secured" | relative_url }}).

## How does international interoperability work?

Each receiving device records the inner payload along with all the other information but may not interpret
its content directly. It is important, therefore, that the issuing country is content with the inner payload
being recorded by a foreign visitors device and when they return home that data could be shared with
the issuing state's systems for contact tracing reasons.

It needs to be shared because it forms part of the contact tracing data, and will be shared with the issuing country
by that country's health authority should the visitor become ill. This allows countries to implement
their own signing and crytographic approach within the inner payload if they wish whilst all using the Herald Protocol.

Worked example (countries mentioned don't necessarily use the Herald protocol):-
- Natalie is from Australia and uses the Victoria contact tracing app
- Jess is from the UK and uses the England app
- Natalie visits the UK and spends 2 hours at a bar talking to Jess. She returns to Australia and falls ill
- Natalie submits her contact information for the past 14 days, that includes Jess' contact tracing payload
  - Victoria state's health authority now has 8 Payloads for Jess, but doesn't know they're for the same person, and a bunch of RSSI information
    - Victoria state's health authority doesn't need to know the make and model of Jess' phone, as she's the one that is exposed
- Victoria state's health authority confirms Natalie is ill, and sends "Hey, a person in my country fell ill on X date, and met someone for your country."
  - This includes the entire contact packet shared by Jess, including the time of receipt.
  - This MAY optionally include 'exposure verifier' data, known only to Natalie and Jess' phones, allowing Jess' phone in a decentralised system, or the England health authority in a centralised system, to verify this data was sent by Jess and received by Natalie. (See the [Secured inner payload]({{"/payload/secured" | relative_url }}) for an example approach)
- England's state health authority processes the shared contact payload from Victoria state and notifies the user of the exposure if necessary (usually if it hits a notifiable threshold in a centralised system, or always notified in a decentralised one)
  - England Confirms the validity of the contact by verifying the inner packet provided by Jess' phone, and validating the signature of the whole packet
  - In a centralised system, the ClientID would be associated with a particular unique ID, and so England would know it's a single person, Jess, exposed for 2 hours and thus needs notifying. If contact details are already known, a manual contact tracing team could make contact. Alternatively (or as well) a message could be sent to England's app on Jess' phone making her aware, given advice on self isolation and testing, and requesting she submit nearby contact details (See [Second order]({{"/background/glossary" | relative_url }}) contact tracing as to why you'd do this before testing Jess)
  - In a decentralised system, the entire payload, or a decoded portion thereof, could be sent to all phones, and distance estimation and exposure could be calculated locally on Jess' phone. Ideally the inner payload would be linked via time (E.g. TOTP) to the client ID, the client ID would be predictable only by Jess' phone, and the inner payload verifies the time in an encrypted form, preventing replay and relay attacks
