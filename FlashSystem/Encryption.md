---
title: CLI Encryption  & Recovery Key Enablement 
description: 
published: true
date: 2025-08-14T03:51:53.888Z
tags: encryption
editor: markdown
dateCreated: 2024-11-18T22:49:53.159Z
---

## CLI USB Encryption Enablement

The following are steps to use CLI to enable encryption on IBM FlashSystem. 

Process only includes cli setup for USB enablement. Key server setup will be added at a later time. 

In short the process is to enable, prepare, and then commit.  
 

---

> *Ensure the following requirements are met before setting up*

-   At least 3 USB sticks are available
-   License Key has been applied to the FlashSystem
-   No pools exist on the system.

##### USB Encryption Enablement

1\. SSH into cluster 

2\. Verify license has been activated

list encryption  
`lsencryption`

output: (line 1)

```plaintext
status licensed
error_sequence_number
usb_rekey no_key
usb_key_copies 0
usb_key_filename
usb_rekey_filename
keyserver_status licensed
keyserver_rekey no_key
keyserver_pmk_uid
keyserver_pmk_rekey_uid
recovery_key_status licensed
recovery_key_rekey no_key
recovery_key_name
recovery_key_rekey_name
```

3\. Verify USB are recognized

list usb port  
`lsportusb`

output: (status ‘active’)

```plaintext
id node_id node_name node_side port_id status encryption_state service_state
0  1       node1     rear      1       active
8  2       node2     rear      1       active
```

4\. Enable encryption for USB

chencryption  
`chencryption -usb enable`

output:   
no response given back

5\. Verify USB encryption is Enabled

list encryption  
`lsencryption`

output: (line 1)

```plaintext
status enabled
error_sequence_number
usb_rekey no_key
usb_key_copies 0
usb_key_filename
usb_rekey_filename
keyserver_status licensed
keyserver_rekey no_key
keyserver_pmk_uid
keyserver_pmk_rekey_uid
recovery_key_status licensed
recovery_key_rekey no_key
recovery_key_name
recovery_key_rekey_name
```

6\. Generate encryption key & prepare(writes key to usb)

chencryption  
`chencryption -usb newkey -key prepare`

output:  
no response given back

7\. Verify key filename and number of usb copies.   
     *note: Remember a total of at least 3 usb copies is needed for enablement.* 

list encryption  
`lsencryption`

output: (line 3 & 4)

```plaintext
status enabled
error_sequence_number
usb_rekey prepared
usb_key_copies 2
usb_key_filename
usb_rekey_filename encryptionkey_00000204A0605F7E_A0605F7E00000001_FS5200
keyserver_status licensed
keyserver_rekey no_key
keyserver_pmk_uid
keyserver_pmk_rekey_uid
recovery_key_status licensed
recovery_key_rekey no_key
recovery_key_name
recovery_key_rekey_name
```

  
8\. Remove one usb stick and replace with the required 3rd one.   
 

9\. Verify key filename and number of usb copies. Listing should now show a total of 3 copies.  

list encryption  
`lsencryption`

output: (line 4)

```plaintext
status enabled
error_sequence_number
usb_rekey prepared
usb_key_copies 3
usb_key_filename
usb_rekey_filename encryptionkey_00000204A0605F7E_A0605F7E00000001_FS5200
keyserver_status licensed
keyserver_rekey no_key
keyserver_pmk_uid
keyserver_pmk_rekey_uid
recovery_key_status licensed
recovery_key_rekey no_key
recovery_key_name
recovery_key_rekey_name
```

10\. Commit new encryption key to the system

chencryption  
`chencryption -usb newkey -key commit`

output:  
no response given back  
 

11\.  Verify usb\_rekey is now set ‘no’ which means enablement was successful

list encryption  
`lsencryption`

output: (line 3)

```plaintext
status enabled
error_sequence_number
usb_rekey no
usb_key_copies 3
usb_key_filename encryptionkey_00000204A0605F7E_A0605F7E00000001_FS5200
usb_rekey_filename
keyserver_status licensed
keyserver_rekey no_key
keyserver_pmk_uid
keyserver_pmk_rekey_uid
recovery_key_status licensed
recovery_key_rekey no_key
recovery_key_name
recovery_key_rekey_namee
```

## CLI Recovery Key Enablement

In later releases of SVC the system allows for recovery keys to be generate in event of losing the usb sticks and/or the key server is no longer accessible.   
The following outlines the steps needed to create a recovery key with cli. 

In short the process is to enable, prepare, confirm and then commit.  
 

---

1\. SSH into cluster 

2\. Enable recovery key

chencryption  
`chencryption -recoverykey enable`

output:   
no response given back

3\.  Generate a new recovery key

chencryption  
`chencryption -recoverykey newkey -key prepare`

output:   
`CMMVC1127I The new recovery key for the system is: q0Fzb8Mke0MTW/I+dx1tpIE1Jf4iaJ2Sa4dnmjqvuB0=`  
  
***NOTE:** Copy recovery and store in safe place*. 

4\.  Verify recovery status is enabled, rekey is in prepared state and recovery key rekey name exists. 

list encryption  
`lsencryption`

output: (lines 11, 12, & 14)

```plaintext
status enabled
error_sequence_number
usb_rekey no
usb_key_copies 3
usb_key_filename encryptionkey_00000204A0605F7E_A0605F7E00000001_FS5200
usb_rekey_filename
keyserver_status licensed
keyserver_rekey no_key
keyserver_pmk_uid
keyserver_pmk_rekey_uid
recovery_key_status enabled
recovery_key_rekey prepared
recovery_key_name
recovery_key_rekey_name recoverykey_00000204A0605F7E_A0605F7E00000001_FS5200
```

5\. Confirm new recovery key

chencryption  
`chencryption -recoverykey newkey -key confirm`

output: (enter recovery key at prompt)

```plaintext
Enter the new recovery key for the system:

Success: CMMVC1153I The recovery key has been entered correctly.
```

6\. Validate recovery key rekey

lsencryption  
`lsencryption`

output: (line 12)

```plaintext
status enabled
error_sequence_number
usb_rekey no
usb_key_copies 3
usb_key_filename encryptionkey_00000204A0605F7E_A0605F7E00000001_FS5200
usb_rekey_filename
keyserver_status licensed
keyserver_rekey no_key
keyserver_pmk_uid
keyserver_pmk_rekey_uid
recovery_key_status enabled
recovery_key_rekey confirmed
recovery_key_name
recovery_key_rekey_name recoverykey_00000204A0605F7E_A0605F7E00000001_FS5200
```

7\. Commit new recovery key to the system

chencryption  
`chencryption -recoverykey newkey -key commit`

output:  
no response given back

8\. Verify recovery key items

-   recovery key rekey is ‘no’
-   recovery key name exists
-   recovery key rekey name is no longer present

lsencryption  
`lsencryption`

output: (line 12,13, & 14)

```plaintext
status enabled
error_sequence_number
usb_rekey no
usb_key_copies 3
usb_key_filename encryptionkey_00000204A0C05FB8_A0C05FB800000001_FS5200_DR
usb_rekey_filename
keyserver_status licensed
keyserver_rekey no_key
keyserver_pmk_uid
keyserver_pmk_rekey_uid
recovery_key_status enabled
recovery_key_rekey no
recovery_key_name recoverykey_00000204A0C05FB8_A0C05FB800000001_FS5200_DR
recovery_key_rekey_name
```

## FlashSystem Info(Lab)

Encryption License:

|     |     |     |
| --- | --- | --- |
| 11/18/24 | FS5200\_DR (78F1CL6) | 6FCE-9FDB-18A9-161B |
| 11/18/24 | FS5200        (78F1CKP) | 58F5-EBA6-FB12-535F |

Recovery Key:

|     |     |     |
| --- | --- | --- |
| 11/18/24 | FS5200\_DR (78F1CL6) | Clustered |
| 11/18/24 | FS5200        (78F1CKP) | q0Fzb8Mke0MTW/I+dx1tpIE1Jf4iaJ2Sa4dnmjqvuB0= |