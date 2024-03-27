# Sysmon, Suricata, Splunk

For this lab we will install Sysmon and Suricata to then forward the logs to Splunk

Prerequisite: Splunk server, splunk forwarder installed

Note: Suricata is installed on a linux os and just sends the logs as raw data which might not be the most suitable solution, but it gives a great insite on how it works on Linux for educational purpose.

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
