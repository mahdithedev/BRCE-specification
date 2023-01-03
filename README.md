# BRCE protocol 

This README defines the BRCE (Bridge Remote Code Execution) protocol. a TCP based used for executing code
on a target machine that is not available on the public internet, the following document specefies the
0.1.0 version of this protocol. this is not the final specification. newer versions will update this specification and no new specification will be released

# Why

This protocol is designed to execute code on systems wich are unavailible on the internet or any network . Why may we want to do that? the answer to this question is up to the reader my reasons can be different from yours

# Terminology

the communication has 3 components a Client component a Bridge component and a Target component.
the following is the definition of each and their role in the communication

| Component | Role |
| ------------------ | ---- |
| Client | this component sends command requests to the bridge |
| Bridge             | as the name states it works like a proxy between the target and the client |
| Target             | this is the machine that executes the command requests received from the bridge and sends output requests to the bridge |

## Other terminology

online target: a target that established a connection with the Bridge component

channel: the connection between the client and the Bridge and the connection between the Target and the Bridge as a whole is called a channel  

secure channel: in this type of channel the data is encrypted by the client and is only decrypted by the target

profile: a set of information about the target

# Protocol overview

The protocol has three steps

1. a Target machine must establish a connection with the Bridge to change its status to online. 
2. the Client sends a connection request that initiates a session in the Bridge this session can contain Client authentication data and will manage the channel.
3. the client can send arbitrary payload along with its size to the target (to the Bridge and finally the Bridge sends it to the target)

While the plain version of this protocol is simple, it can be enhanced by adding authentication and encryption.

# Packet structure

a packet contains three parts a type, size, and a payload

the type as the name states shows the packet purpose (e.g. establishing a connection), the size tells the size of the payload and the payload is arbitrary data

| Type | Size | Payload |
| ---- | ---- | ------- |
| 1 byte | 2 bytes | N bytes |

## example

```python
packet = b"\x03\x00\x06ls -l\x0D"
```

## Types

| Type | Code | Description |
| ---- | ------- | -------- |
| PROF | 0 | will create a profile on the bridge |
| INIT | 1 | establishes a connection between the target and the bridge |
| CONN | 2 | initializes a session in the Bridge and a channel between the Client and the Target |
| LIS  | 5 | if a client sends a LIS packet it means that it is requesting the list of profiles and if a server is sending this packet the payload contains the list of profiles |
| INP  | 4 | input request |
| OUT  | 5 | output request |
| ERR  | 6 | contains information about an error that occurred |

the second table defines what the payload represents in each packet type

| Type | Payload |
| ---- | ------- |
| PROF | a json structure containing all profile info |
| INIT | Profile Id |
| CONN | Profile Id |
| LIS  | an expression that will be matched against profiles, the matched profiles will be sent in a packet with the same type |
| INP  | arbitrary input |
| OUT  | interpeter output |
| ERR  | error code |

## Size

the size of the payload the value must be equal to or greater than 0 and equal or smaller than 
2^16 -1 (65,535)

## Payload 

contains arbitrary data

# Profiles

a profile is a filed-value structure stored in a JSON file that contains info about the target

| Field | Value |
| ----- | ----- |
| ID | profiles unique identifier |
| OS | operating system software |
| OS Version | operating system version |
| Interpreter | The program that interprets the input request | 
| WHO | contains information about the job of this system (e.g. My personal computer) |

# Errors

at first, this protocol does not handle any payload interpretation error so if you send a bash command but the target interprets the payload using windows shell this is not detected by the protocol and will only be an output error at the client side

you can find info about each error in the table below

| Code | Name | Description |
| ---- | ---- | ----------- |
| 0 | Target offline | |
| 1 | Profile ID is not valid | raised if a CONN request wants to connect to a profule that doesn't exist |
| 2 | The channel is not created yet | |
| 3 | access denied | if a client is not authenticated then they will receive this error |
| 4 | unknown | some unhandeled error happend in the process |
