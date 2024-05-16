---
title: Block Telnet on Brocade Switches
description: 
published: true
date: 2023-06-08T13:38:42.787Z
tags: 
editor: markdown
dateCreated: 2023-06-08T13:30:29.467Z
---

1. Log into the switch with admin permissions
2. Clone the default policy
```
ipfilter --clone BlockTelnet -from default_ipv4`
```
3. Save the new policy
```
ipfilter --save BlockTelnet
```
4. Verify the new policy exists
```
ipfilter --show
```
5. Add a rule to the policy to block telnet (port 23)
```
ipfilter --addrule BlockTelnet -rule 1 -sip any -dp 23 -proto tcp -act deny
```
6. Save the new policy
```
ipfilter --save BlockTelnet
```
7. Verify the new policy has been updated
```
ipfilter --show
```
8. Activate the new policy (this will effectivtely block port 23)
```
ipfilter --activate BlockTelnet
```
9. Verify the new policy is active (the default_ipv4 policy should be displayed as defined
```
ipfilter --show
```
```
Name: default_ipv4, Type: ipv4, State: defined
Rule    Source IP                               Protocol   Dest Port   Action
1     any                                            tcp       22     permit
2     any                                            tcp       23     permit
3     any                                            tcp       80     permit
4     any                                            tcp      443     permit
5     any                                            udp      161     permit
6     any                                            udp      123     permit
7     any                                            tcp      600 - 1023     permit
8     any                                            udp      600 - 1023     permit

Name: default_ipv6, Type: ipv6, State: active
Rule    Source IP                               Protocol   Dest Port   Action
1     any                                            tcp       22     permit
2     any                                            tcp       23     permit
3     any                                            tcp       80     permit
4     any                                            tcp      443     permit
5     any                                            udp      161     permit
6     any                                            udp      123     permit
7     any                                            tcp      600 - 1023     permit
8     any                                            udp      600 - 1023     permit

Name: BlockTelnet, Type: ipv4, State: active
Rule    Source IP                               Protocol   Dest Port   Action
1     any                                            tcp       23       deny
2     any                                            tcp       22     permit
3     any                                            tcp       23     permit
4     any                                            tcp       80     permit
5     any                                            tcp      443     permit
6     any                                            udp      161     permit
7     any                                            udp      123     permit
8     any                                            tcp      600 - 1023     permit
9     any                                            udp      600 - 1023     permit
```