- Start Date: 2020-02-18
- HIP PR: <!-- leave this empty -->
- Tracking Issue: <!-- leave this empty -->

# Summary
[summary]: #summary

LongFi is not a full protocol from the ground up, but instead a blockchain layer on top of LoRaWAN. This allows any off-the-shelf LoRaWAN device to connect to the Helium network if you can update its AppKey and AppEui.

# Motivation
[motivation]: #motivation

There are many LoRaWAN compatible devices already out there and LoRaWAN already has many desirable protocol features (ACK, downlink, FCC certified, international definition). In order to accelerate adoption of the Helium network and to lower technical barriers, LongFi is no longer a distinct protocol from LoRaWAN but instead a layering of some blockchain components on top of LoRaWAN. 


# Stakeholders
[stakeholders]: #stakeholders

* LoRaWAN device users

# Detailed Explanation
[detailed-explanation]: #detailed-explanation

This initial implementation of LongFi on LoRaWAN focuses on a single method of end-device activation: Over-the-Air Activation (OTAA). Activation by Personalization (ABP) is currently not supported.

The Helium network will operate on LoRaWAN channels 48-55 (sub-band 7):
```
Channel 48, 911.900 Mhz
Channel 49, 912.100 Mhz
Channel 50, 912.300 Mhz
Channel 51, 912.500 Mhz
Channel 52, 912.700 Mhz
Channel 53, 912.900 Mhz
Channel 54, 913.100 Mhz
Channel 55, 913.300 Mhz
```

In OTAA, DevEUI, AppEUI, and AppKey are all the unencrypted LoRaWAN primitives used in the Join Request; DevEUI is currently ignored by LongFi.

```
___________________________________________________________
|   Size (octets)  |      8     |      8     |      8     |
|   Join Request   |   AppEUI   |   DevEUI   |  DevNonce  |
‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾
```
source: LoRaWAN Specification 6.2.4 

The LongFi primitives of Organizational Unique Identifier (OUI) and DeviceId are mapped into AppEUI. The most significant 2 octets are OUI while the least significant octets are DeviceId.

```
_______________________________________________
|               AppEui (8 octets)             |
|     OUI (4 octets)   | Device_ID (2 octets) |
‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾
```

Thus Helium miners receiving these unencrypted Join messages are able to route the request to the appropriate Router/NetworkServer by deriving the OUI from the AppEui and then looking up the OUI route from the blockchain records.

Assuming the OUI is registered appropriately, hotspots will route the JoinRequest packet to the appropriate Router/NetworkServer. The Router/NetworkServer will use the Message Integrity Check (MIC) to authenticate the JoinRequest and, if successful, an unencryptd JoinAccept message will be communicated down to the hotspot and then the device, providing a NetId, DevAddr, and AppNonce. 

```
_______________________________________________________________________________________________
|   Size (octets)  |      3     |    3    |    4    |      1       |    1     | (16) Optional |
|   Join Accept    |   AppNonce |  NetId  | DevAddr |  DLSettings  | RxDelay  |   CFList      |
‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾
```
source: LoRaWAN Specification 6.2.5 

Combined with the AppKey, these fields allow the Device and Router/NetworkServer to derive the same NwkSKey and AppSKey (LoRaWAN Specification 6.2.5). Henceforth, payloads are encrypted using NwkSkey and AppSkey (LoRaWAN Specification 4.3.3).

The DevAddr is used by LongFi to indicate the OUI and this will be part of the Frame Header Structure (FHDR) of all messages after the successful Join; this enables hotspots to continue forwarding packets to the appropriate Router/NetworkServer.

The Router/NetworkServer derives the DeviceID by bruteforcing the MIC.

The Channel Frequency List (CFList) is included to configure devices only to use the Helium sub-band 7.

# Drawbacks
[drawbacks]: #drawbacks

- Devices are requried to update their AppKey and AppEui
- There is no "Fingerprint" mechanism; that is to say, there no way for a Server/NetworkServer to validate a message before accepting it from a Hotspot; therefore, Hotspot will forward packets without any negotiation with OUI owners

# Rationale and Alternatives
[alternatives]: #rationale-and-alternatives

- A standalone LongFi protocol was taking too long
- Modifications to the LoRaWAN specification may be made later to try to address any architectural shortcomings

# Unresolved Questions
[unresolved]: #unresolved-questions


# Deployment Impact
[deployment-impact]: #deployment-impact


# Success Metrics
[success-metrics]: #success-metrics
