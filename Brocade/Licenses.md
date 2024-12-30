---
title: Brocade Licenses
description: 
published: true
date: 2024-12-30T21:43:36.230Z
tags: san, brocade, license
editor: markdown
dateCreated: 2022-02-15T19:23:50.948Z
---

# Brocade SAN Switch Licenses
In order to activate a license, you must have a **Transaction Key** for each license to be activated along with the **License ID** (LID) of the appropriate switch.
## Obtaining the Transaction Key
A Transaction key is unique key, along with the LID, used to generate a software license from the Broadcom licensing portal. The transaction key is issued when a license is purchased. The transaction key is delivered in one of two methods:
- **Paperpack** – The transaction key is printed and delivered within a POD Optic Hardware Kit. 
- **E-license** – The transaction key is contained in an email that is sent instantly to the customer after the sales order is created. The customer is sent the email message within a few minutes after the sales order is submitted, although the timing will vary depending on the network, internet connection, and so on. 
## Obtaining the License ID
You must have a license ID to generate the license file in the Broadcom licensing portal.
Obtain the license ID of the switch or chassis using the CLI method: 
1.	Connect to the switch, and log in using an account with admin permissions. 
2.	Enter the following command to display the license ID of the switch:
```
license --show -lid
```
## Generating a License Key 
Use the following procedure to generate and obtain a FOS license key. 
1.	Go to https://www.broadcom.com, and then select the LOGIN drop-down at the top-right of the web page. 
2.	Click **LOGIN** or **REGISTER**. Once logged in, you will be redirected to the Broadcom support portal. 
3.	Click **Brocade Products**. You will be redirected to the Brocade Products page. 
4.	Click **Licensing**. You will be redirected to the Broadcom Licensing Portal page. 
5.	Enter the transaction key. Click Next to continue. 
> Re-host keys are generated only by the SANnav application and are used only on SANnav. 
{.is-info}
6.	Enter the license ID (LID) that you obtained earlier in the Unit Information field. Click **Next** to continue. 
7.	Read the Broadcom End User License Agreement, and if you agree to the terms, select the **I have read and accept** checkbox. 
8.	Click **Generate** to generate the license. 
9.	The license key will be sent by email and can also be downloaded by selecting the blue license hyperlink from the Broadcom licensing portal UI. 
10.	If a license string is generated, save it to your local folder for future reference. 
11.	If an XML certificate file is generated, save it to the remote server, where it will be retrieved for installing the license. 
## Installing the License Using a License String 
For Brocade Gen 6 switches and directors, you must pass the license key in the license --install command to install the licenses. 
1.	Connect to the switch, and log in using an account with admin permissions. 
2.	Activate the license using the license --install -key <lic_key> command. 
```
license --install -key HP9ttZNSgmB4MCD3NmNWgQDWtAKBFtXtBSFJF
```
3.	Verify that the license was installed by entering the license --show command. The licensed features that are currently installed on the switch are listed. If the feature is not listed, use the license --install command to install the license. 
```
license --show
```

## Displaying Port License Assignments 
When you display the available licenses, you can also view the current port assignment of those licenses. 
Use the following procedure to display the port license assignments. 
1.	Connect to the switch, and log in using an account with admin permissions. 
2.	Enter the license --show -port command to view how port licenses are currently assigned.
```
license --show -port
```

## Static Port Assignment
When Dynamic Ports On Demand (DPOD) is enabled, ports are licensed from a pool of available licenses based on the server blade or switch installation.  Each port that is enabled will secure a license on a first-come, first-served basis.  If you would like to statically assign port licenses, use the following command.  Example below would reserve licenses on the first 16 ports in a switch.
```
license --reserve -port 0-15
```

Use the command in the previous section to display the port licenses after making the static assignment to ensure they were properly assigned.