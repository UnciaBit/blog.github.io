# Useful tools for CTFs and security analysis

## SNMP Enumeration

[snmpwalk](https://linux.die.net/man/1/snmpwalk)
[How to use snmpwalk](https://www.comparitech.com/net-admin/snmpwalk-examples-windows-linux/)
[Other Guide]https://fareedfauzi.gitbook.io/oscp-notes/services-enumeration/snmp
### Example Usage
```bash
snmpwalk -v1 -c public <ip address>
```