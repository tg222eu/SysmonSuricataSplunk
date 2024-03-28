# Sysmon, Suricata, Splunk

For this lab we will install Sysmon and Suricata to then forward the logs to Splunk

Prerequisite: Splunk server, Splunk Forwarder installed on the sysmon machine

Note: Suricata is installed on a linux os and just sends the logs as raw data which might not be the most suitable solution, but it gives a great insite on how it works on Linux for educational purpose. The eve.json file will not be useful in Splunk

The lab is divided into 5 sections:

1. Installing sysmon
2. Installing suricata
3. Configure Suricata
4. Creating your first rule
5. Send Suricata logs to Splunk

# Sysmon
Sysmon is a great complement and extends windows event logs and is especially useful for threat hunting in an enviroment that does not monitor with EDR.

![alt text](https://github.com/tg222eu/SysmonSuricataSplunk/blob/main/sysmoninstall.png)
```
sysmon64.exe -accepteula -i
```
This will install sysmon and start logging. Its that easy. If you want to use a custom xml file you will need to include it in the parameter
```
sysmon64.exe -accepteula -i sysmonconfig-export.xml
```
If a custom xml is used then I recommend SwiftOnSecurity configuration file as, it filters out unnecessary noise its consider a best practice and then can be further customized according to the enterprise needs. The config can be found here: https://github.com/SwiftOnSecurity/sysmon-config

![alt text](https://github.com/tg222eu/SysmonSuricataSplunk/blob/main/windowsevent.png)

In the windows event viewer the sysmon logs will be stored under Applications and services/Microsoft/Windows/Sysmon

Install splunk forwarder on the machine with sysmon. You can also install "Splunk Add-on for microsoft sysmon" on your server for enhanced field mapping. (s Once the forwarder is installed on the sysmon machine there will be a configuration file which need to be edited in order to index the sysmon logs correctly once sent.

Open up the config file for splunkuniversal forwarder, which is usually installed at c:\Program files\splunkuniversalforwarder\etc\apps\Splunkuniveralworfarder\local\inputs.conf

Add the following text
```
[WinEventLog://Microsoft-Windows-Sysmon/Operational]
checkpointInterval = 5
current_only = 0
disabled = 0
start_from = oldest
```
You can restart splunkforwarder in services and the logs should appear immediately on the splunk server. If no logs appear you can troubleshoot by looking in the logs for clues about errors in splinkd.log. c:\Program files\splunkuniversalforwarder\var\log\splunk\splunkd.log

![alt text](https://github.com/tg222eu/SysmonSuricataSplunk/blob/main/SplunkSysmon.png)

Here we can see XmlWinEventLog which is our sysmon logs

# Suricata

Install Suricata on a Ubuntu server with the commands below
```
sudo add-apt-repository ppa:oisf/suricata-stable
sudo apt-get update
sudo apt-get install suricata
```
![alt text](https://github.com/tg222eu/SysmonSuricataSplunk/blob/main/pictures/suricataversion.png)

You can check the version just to see if its working as it should
```
suricata-update check-versions
```

Enable Suricata on startup. Remember to stop or restart the service in when changing configuration
```
sudo systemctl enable suricata.service
```
Useful commands
```
sudo systemctl start suricata
sudo systemctl stop suricata
```
# Configure Suricata

All configuration are stored in the suricata.yaml file. This file is huge, but we only need to configure IP and interfaces.

Open up the configuration file
```
sudo vim /etc/suricata/suricata.yaml
```
![alt text](https://github.com/tg222eu/SysmonSuricataSplunk/blob/main/pictures/suricatayaml.png) <br>
By typing "/" you can search for "HOME_NET". There you have to change the IP number to the network subnet you want to monitor. In my case its 192.168.4.0/24.

Its likely that the interface is not set to the machines inferface and has to be added. Search for /af-packet within VIM and check to make sure the interface is correct. The interface should match the name of the interface thats shown up in “ifconfig”. example my case was ens160

Optionally its recommended to change the interface for the pcap section aswell and enable community-id to true.

![alt text](https://github.com/tg222eu/SysmonSuricataSplunk/blob/main/pictures/suricatayaml.png)
We need to update Suricata in order for it to create additional configuration and rule files. Right now they are missing. When updating and it detects no rules selected and it will choose the default, the “Emerging Threats Open” ruleset will be used. rules.emergingthreats.net.
```
sudo suricata-update
```
![alt text](https://github.com/tg222eu/SysmonSuricataSplunk/blob/main/pictures/suricataruleset.png)
Suricata do provide a default ruleset. This ruleset monitors a lot of data and can be filtered out further. There are other ruleset included that you can use which is often created by open source projects or for commercial use that require subscription. You can check the available rules by typing
```
sudo suricata-update list-sources
```
To use another ruleset rather then the default one, we can use “tgreen/hunting” as example, use the command below

sudo suricata-update enable-source tgreen/hunting

![alt text](https://github.com/tg222eu/SysmonSuricataSplunk/blob/main/pictures/suricataruletest.png)

To test the rules you can run a test with command below and check if the rules load successfully or fails, or check for any other issues
```
sudo suricata -T -c /etc/suricata/suricata.yaml -v
```

The actual intrusion logs are stored in /var/log/suricata/fast.log
eve.json are the same but in json format

![alt text](https://github.com/tg222eu/SysmonSuricataSplunk/blob/main/pictures/testids.png)

To test the IDS type command below the check the result in fast.log
```
curl http://testmynids.org/uid/index.html
```
# Create your first alert rule

Lets create a custom ICMP alert rule

Some rules are stored in /etc/suricata/rules in Linux. For ubuntu its usr/share/suricata/rules
```
sudo vim /usr/share/suricata/rules/local.rules
```
![alt text](https://github.com/tg222eu/SysmonSuricataSplunk/blob/main/pictures/localrule.png)

Write down your first rule. This rule with monitor from any to any network
```
alert icmp any any -> $HOME_NET any (msg:”ICMP Ping”; sid:1; rev:1;)
```
Now we have the rule local.rules, we need to configure suricata to read the custom rule file. Open the suricata.yaml configuration file
```
sudo vim /etc/suricata/suricata.yaml
```
Search for /rule-path

Update the rule path as /usr/share/suricata/rules/local.rules where you created the file. To make sure the rule is loaded properly, restart the suricata service in systemctl.

Test ping from a client that is monitored by suricata and see if it generates logs

![alt text](https://github.com/tg222eu/SysmonSuricataSplunk/blob/main/pictures/succesfulrule.png)

If this dosnt go through, a really useful command to check if there is any syntax error in the rule. Run command below and replace ens160 with your own interface
```
sudo suricata -c /etc/suricata/suricata.yaml -i ens160
```
![alt text](https://github.com/tg222eu/SysmonSuricataSplunk/blob/main/pictures/ICMPping.png)<br>
Onces everything runs as it should there should be a ICMP alert from the rule we just created in the fast.log file. 

# Suricata to Splunk

Install Suricata Universal Forwarder on the suricata machine
```
wget -O splunkforwarder-9.2.0.1-d8ae995bf219-linux-2.6-amd64.deb "https://download.splunk.com/products/universalforwarder/releases/9.2.0.1/linux/splunkforwarder-9.2.0.1-d8ae995bf219-linux-2.6-amd64.deb"
sudo dpkg -i <package path>
```
Start Splunk
```
cd /opt/splunkforwarder/bin/
sudo ./splunk start –accept-license
```
Set admin and password, then make it start on boot
```
sudo systemctl start SplunkForwarder
```
Configure the IP address of Splunk server
```
./splunk add forward-server [IP address of splunk server:9997]
```
Choose what log files/folder from the Ubuntu Server should be sent to Splunk
```
./splunk add monitor -auth username:password /var/log/suricata
```
