# Sysmon, Suricata, Splunk

For this lab we will install Sysmon and Suricata to then forward the logs to Splunk

# Sysmon
Sysmon is a great complement and extends windows event logs and is especially useful for threat hunting in an enviroment that does not monitor with EDR.

![alt text](https://github.com/tg222eu/SysmonSuricataSplunk/blob/main/sysmoninstall.png)

sysmon64.exe -accepteula -i sysmonconfig-export.xml
Uses a custom config xml file, otherwise it will run a default one. SwiftOnSEcurity is the best practice to use at start then customize according to the enterprise from there. This filter away the unnecessary noise of events
