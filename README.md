# tardfri_pi
## Brief
The Tardfri can be controlled by using COAP protocol. Users can use the [libcoap](https://github.com/obgm/libcoap) and coap-client to access the devices. However, even though the COAP is an opened protocol, still some private fields and methods are adapted in the IKEA Tardfri. This document show the setup of the libcoap and simple testing procedures.

## Hardwares and Softwares
### Hardware:
* PI 4B (raspbian 10 buster)
* Tardfri Light (firmware 2.3.050)
* Tardfri remote (firmware 2.3.014)
* Tardfri Gateway (firmware 1.12.34)

### Software:
* libcoap (v4.1.2 tag coap-client)

## Installation
Reference: 
> https://learn.pimoroni.com/tutorial/sandyj/controlling-ikea-tradfri-lights-from-your-pi

1. *ssh to the pi*
1. *sudo apt-get install build-essential autoconf automake libtool*
1. *git clone https://github.com/obgm/libcoap.git*
1. *cd libcoap*
1. *git checkout coap-tinydtls*
1. *git submodule update --init --recursive*
1. *./autogen.sh*
1. *./configure --disable-documentation --disable-shared*
1. *make*
1. *sudo make install*
1. *coap-client*, should shows v4.1.2 is installed

## Command

Reference:
> https://twitter.com/home_assistant/status/925373865802502144
> https://github.com/glenndehaan/ikea-tradfri-coap-docs (this provides a comprehensive commands explanation and code definitions) 

### Register user on gateway and get the preshared key
1. Drop down the secure code on the back of the gateway
1. Get the gateway ip, which can be found on the admin page of router with hostname GW-<MAC address>
1. *coap-client -u "Client_identity" -k "<secure code>" -v 0 -e '{"9090":"<username>"}' -m POST "coaps://<gateway_ip>:5684/15011/9063"*
1. the message *{"9091":"<preshared_key>","9029":"<firmware version>"}* should be returned

The <username> and <preshared_key> is used for furhter controls the Tradfri devices
  
### List devices
* *coap-client -m get -u "<username>" -k "<preshared_key>" "coaps://<gateway_ip>:5684/15001"*
* response: *[<device_id>, ....]*

### List groups
* *coap-client -m get -u "<username>" -k "<preshared_key>" "coaps://<gateway_ip>:5684/15004"*
* response: *[<group_id>, ....]*
  
### Get device status
* *coap-client -m get -u "<username>" -k "<preshared_key>" "coaps://<gateway_ip>:5684/15001/65538"*
* response: *[\<code\>, \<value\> ....]
* for light bud, the message returns, *{"9001":"TRADFRI bulb","9003":65539,"9002":1607509104,"9020":1610157124,"9054":0,"9019":1,"3":{"0":"IKEA of Sweden","6":1,"1":"TRADFRI bulb E27 WS opal 980lm","2":"","3":"2.3.050"},"5750":2,"3311":[{"5850":0,"5851":91,"5717":0,"5711":250,"5709":24841,"5710":24593,"5706":"f5faf6","9003":0}]}*
  
### Control device
* *coap-client -m put -u "<username>" -k "<preshared_key>" -e '{"\<code\>":[{"<\code\>":<value>}]}' "coaps://<gateway_ip>:5684/15001/<device_id>"*
* e.g. turn the light on * *coap-client -m put -u "<username>" -k "<preshared_key>" -e '{"3311":[{"5850":1}]}' "coaps://<gateway_ip>:5684/15001/<device_id>"*
* e.g. turn the light off * *coap-client -m put -u "<username>" -k "<preshared_key>" -e '{"3311":[{"5850":0}]}' "coaps://<gateway_ip>:5684/15001/<device_id>"*

### Control group
* similar to control device but endpoints to 15004, i.e. * *coap-client -m put -u "<username>" -k "<preshared_key>" -e '{"\<code\>":[{"<\code\>":<value>}]}' "coaps://<gateway_ip>:5684/15004/<device_group>"*

