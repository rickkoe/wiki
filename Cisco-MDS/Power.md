---
title: Cisco MDS Power Redundancy
description: 
published: true
date: 2023-05-16T19:38:14.282Z
tags: 
editor: markdown
dateCreated: 2023-05-16T19:38:10.402Z
---

You can configure one of the following power modes to use the combined power provided by the installed power supply units (no power redundancy) or to provide power redundancy when there is power loss. We recommend that you configure the full redundancy power mode on your switch for optimal performance.

Combined mode—This mode uses the combined capacity of all the power supplies. In case of power supply failure, the entire switch can be shut down (depending on the power used) causing traffic disruption. This mode is seldom used, except in cases when the switch requires more power.

Input Source (grid) redundancy mode—This mode allocates half of the power supplies to the available category and the other half to the reserve category. You must use different power supplies for the available and reserve categories so that if the power supplies used for the active power fails, the power supplies used for the reserve power can provide power to the switch. If the grid-redundancy mode is lost, the power mode reverts to combined mode.

Power-supply (N+1) redundancy mode—This mode allocates one power supply as reserve to provide power to the switch in case an active power supply fails. The remaining power supplies are allocated for the available category. The reserve power supply must be at least as powerful as each of the power supplies used for the active power.

Full-redundancy mode—This mode is a combination of input-source (grid) and power-supply (N+1) redundancy modes. Similar to the input-source redundancy mode, this mode allocates half of the power supplies to the available category and the remaining power supplies to reserve category. One of the reserve power supplies can alternatively be used to provide power if a power supply used for the active power fails.

For more information on the power supply modes supported on your switch, see the Hardware Installation Guide corresponding to your switch.