## Introduction

Preparation is key in most industries but for cybersecurity it is a must. Understanding your threats so you can stay one step ahead of your adversaries is important to progress throughout this industry. An effiecient way of gaining this information is by setting up a honeypot. Honeypots are designed to lure attackers and gather valuable information about their tactics. To gain information about diverse variety of vulnerablities (SSH, Malware, Telnet, etc) I will be deploying T-Pot, a multi honeypot system.

## Tools and Methods that I Used

- A virtual machine or physical server with a fresh installation of Debian 11
- PuTTY
- At least 4 GB of RAM, 128 GB of storage, and a stable internet connection.
- Basic knowledge of Linux command-line operations

## Creating the VM in Azure

![Screenshot (130)](https://github.com/user-attachments/assets/0e28e3aa-573c-4db5-892f-8368925c01c5)
T-pot is resource intensive due to the multiple honeypots and the elastic stack(Elasticsearch,Kibana etc) that collects logs and analyses them. For this I chose 16 GiB ram with 4 vCPUs. This can be expensive to run for a few days so be aware.

## Configuring the VM

Under the Networking setting for your virtual machine, add an inbound rule to cover all possible ports making it easier to attract attackers.
![Untitled](https://github.com/user-attachments/assets/df77627b-c0c8-4309-a581-0348b932fc6a)

Next, we'll configure the VM's boot process. First, download PuTTY, a free SSH client. Open PuTTY Key Generator and load your private key.This creates a public-private key pair for secure authentication. Then, launch PuTTY Configuration. Navigate to Connection > SSH > Auth, browse to and select your newly generated private key file. This allows you to use key-based authentication, which is more secure than password-based login . Finally, click "Open" to establish a secure connection to your VM.
![Untitled(1)](https://github.com/user-attachments/assets/73e82d3c-ffe7-4557-aaa6-af2292fe20c3)
![Untitled(2)](https://github.com/user-attachments/assets/a9cc164d-37fd-45f7-8466-70ee4a63692c)


## Installing the honeypot

```jsx
sudo apt update
sudo apt upgrade -y
sudo apt install git
sudo git clone https://github.com/telekom-security/tpotce
sudo cd tpotce/iso/installer/
sudo ./install.sh --type=user
```

The installation script will guide you through the process, including setting up a password for the web interface and choosing honeypot services.

Once T-Pot is installed, it’s important to verify that all services are running smoothly:

```
sudo apt update && sudo apt upgrade -y
sudo docker-compose pull
sudo systemctl restart tpot
```

Whilst checking the status of my T-Pot I came across an error.

```jsx
Jun 22 19:07:16 HoneyPot docker[248837]: tpotinit             |
Jun 22 19:07:16 HoneyPot docker[248837]: tpotinit             | # Error: WEB_USER is not set 
```

The web user interface should be set up during the installation but if thsat fails you can manually set up your password and login to access the web interface. This involves editing the enviroment file. Below is the command:

```jsx
sudo nano ~/tpotce/docker/.env
```

The file should look like this:

```jsx
# Set Web usernames and passwords here. This section will be used to create / update the Nginx password file nginxpasswd.
#  <empty>: This is the default
#  <base64 encoded htpasswd usernames / passwords>:
#   Use 'htpasswd -n -b "username" "password" | base64 -w0' to create the WEB_USER if you want to manually deploy T-Pot, run 'install.sh' to automatically add a user during installation, or 'genuser.sh' if you just want to add a web user.
#   Example: 'htpasswd -n -b "tsec" "tsec" | base64 -w0' will print dHNlYzokYXByMSRYUnE2SC5rbiRVRjZQM1VVQmJVNWJUQmNmSGRuUFQxCgo=
#   Copy the string and replace WEB_USER=dHNlYzokYXByMSRYUnE2SC5rbiRVRjZQM1VVQmJVNWJUQmNmSGRuUFQxCgo=
#   Multiple users are possible:
#   WEB_USER=dHNlYzokYXByMSRYUnE2SC5rbiRVRjZQM1VVQmJVNWJUQmNmSGRuUFQxCgo= dHNlYzokYXByMSR6VUFHVWdmOCRROXI3a09CTjFjY3lCeU1DTloyanEvCgo=
WEB_USER=

# Set Logstash Web usernames and passwords here. This section will be used to create / update the Nginx password file lswebpasswd.
# The Lostsash Web usernames are used for T-Pot log ingestion via Logstash, each sensor should have its own user.
#  <empty>: This is empty by default.
#  <'htpasswd encoded usernames / passwords'>:
#   Use 'htpasswd -n -b "username" "password" | base64 -w0' to create the LS_WEB_USER if you want to manually deploy the sensor.
#   Example: 'htpasswd -n -b "sensor" "sensor" | base64 -w0' will print c2Vuc29yOiRhcHIxJGVpMHdzUmdYJHNyWHF4UG53ZzZqWUc3aEFaUWxrWDEKCg==
#   Copy the string and replace / add LS_WEB_USER=c2Vuc29yOiRhcHIxJGVpMHdzUmdYJHNyWHF4UG53ZzZqWUc3aEFaUWxrWDEKCg==
#   Multiple users are possible:
#   LS_WEB_USER=c2Vuc29yMTokYXByMSQ5aXhNRk5yMCR6d3F2dGFwQ2x0cFBhU1pqMm9ZemYxCgo= c2Vuc29yMjokYXByMSRtYTlOS1J2NCQvU3dsVVBMeW5RaVIyM3pyWVAzOUkwCgo=
LS_WEB_USER=
```

After following the instructions you should be able to access the web interface. 

## Interface

After running my honeypot for a week, I accessed the web interface to review the collected data. The interface provides an overview of the activities and logs recorded during this period. It includes real-time monitoring and detailed insights into various aspects of the attacks.

![Screenshot (132)](https://github.com/user-attachments/assets/6a4d879d-3e98-4f27-a335-217049f27105)

## Attack Map

The attack map is a crucial feature that visualizes ongoing attacks on the honeypot. It displays real-time data on the geographical origins of the attacks, highlighting the countries from which the highest number of attacks originate. The map also differentiates the types of attacks using various colors, helping in identifying patterns and potential threats.

Below the map, there are detailed logs listing the IP addresses involved and the corresponding attack types. This section is instrumental in understanding the attack landscape and assessing the severity of each incident.

![Screenshot (108)](https://github.com/user-attachments/assets/00b53c36-8207-4fd0-a3ec-5d8eec3a5c44)


To mitigate risks, suspicious IP addresses can be reported on platforms like [abuseipdb.com](http://abuseipdb.com/), which helps in notifying relevant authorities and potentially blacklisting malicious actors.
![Screenshot (121)](https://github.com/user-attachments/assets/5a4c4230-1397-4fd5-a717-2955590bc7e9)


## ElasticVue Metrics

ElasticVue provides comprehensive metrics on the cluster hosting the honeypot. It includes information such as RAM usage and disk space utilization, which are critical for maintaining optimal performance and avoiding resource bottlenecks.

![5d2be3fe-78e5-4337-8523-e5a4e1122349](https://github.com/user-attachments/assets/d8412b6b-fd98-4244-813c-077a1803ec78)

![Screenshot (109)](https://github.com/user-attachments/assets/603f1635-05e6-42e1-90a1-1f5fe0ec8c90)



The tool also offers insights into system health, including the status of various nodes and indices. On the shards page, users can observe data distribution and storage efficiency, crucial for understanding how the system handles and processes the captured logs.
![Screenshot (110)](https://github.com/user-attachments/assets/2474b44a-fba7-42fb-8c4b-47071176475d)



## Kibana Metrics

Kibana is a powerful tool used to visualize and analyze the data collected by the honeypot. It simplifies complex datasets into understandable charts and graphs, making it easier to identify trends and anomalies. Kibana supports a variety of data visualizations, including time-series analysis, heat maps, and pie charts, each serving different analytical needs.

The visualizations help in quickly pinpointing attack peaks, unusual activity patterns, and the overall volume of traffic hitting the honeypot. This level of analysis is essential for making informed decisions about security measures and response strategies.
![Screenshot (112)](https://github.com/user-attachments/assets/7d94b6b6-1699-4c8f-88f2-65e79634959d)

![Screenshot (113)](https://github.com/user-attachments/assets/fc76bd8e-61a3-4bdc-9ad2-14366f8adf4b)

![Screenshot (114)](https://github.com/user-attachments/assets/1bb849cd-595c-4afc-9752-6f8d1298bd6c)

## Spiderfoot Metrics

Spiderfoot is used to gather more detailed information about specific IP addresses. It can identify the ISPs, domains associated with these IPs, and whether the IP have been reported for malicious activities. This tool provides a comprehensive view of potential threats and helps in profiling the attackers.

By using Spiderfoot, one can enrich the data collected from the honeypot, adding context to the raw IP information. This enriched data is invaluable for understanding the nature of the attacks and for developing targeted mitigation strategies.

![Screenshot (117)](https://github.com/user-attachments/assets/b303d325-c687-4c4e-baeb-18bcaa5f9694)


## Conclusion

In this project, I set up a T-Pot honeypot in Microsoft Azure to gain insights into potential threats and attack patterns. By integrating tools like Kibana, ElasticVue, and Spiderfoot, I could effectively monitor and analyze the collected data. The attack map provided real-time visualization of attack origins, while Kibana offered detailed metrics and trends analysis. ElasticVue ensured optimal performance monitoring of the honeypot system.

To assess the impact of this setup, I observed the honeypot over a week, gathering valuable data on attack frequencies, types, and origins. Spiderfoot provided detailed information on malicious IP addresses, aiding in threat profiling. Reporting these IPs to services like [abuseipdb.com](http://abuseipdb.com/) further contributed to broader cybersecurity efforts.

The data gathered highlighted current threats and helped refine security strategies. It's important to note that while this honeypot setup provides valuable insights, the volume of security events can be influenced by factors such as network usage patterns and the honeypot's visibility to potential attackers.

In summary, this project demonstrates the effective use of a multi-honeypot system integrated with robust analysis tools to enhance threat detection and understanding. As a cybersecurity analyst, I recognize the importance of continuous monitoring and contextual analysis of security events. This approach allows for accurate threat identification and helps in developing and optimizing incident response strategies to combat evolving cyber threats.
