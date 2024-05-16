---
title: Validating a Brocade Switch Installation
description: 
published: true
date: 2023-01-22T19:45:11.119Z
tags: brocade, fibre channel, script, validation
editor: markdown
dateCreated: 2022-02-23T22:11:35.807Z
---

```
chassisshow
switchshow
license --show
wwn
rootaccess --show
firmwareshow
hashow
slotshow
sfpshow
defzone --show
cfgshow
ethif --show eth0
ipaddrshow
```