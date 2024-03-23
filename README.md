# Sysmon, Suricata, Splunk

For this lab we will install Sysmon and Suricata to then forward the logs to Splunk

Prerequisite: Splunk server, splunk forwarder installed

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

Install splunk forwarder on the machine with sysmon. Once installed there will be a configuration file which need to be edited in order to index the sysmon logs correctly once sent. 
