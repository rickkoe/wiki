---
title: LDAP Configuration on IBM FlashSystem
description: 
published: true
date: 2022-04-27T21:35:33.829Z
tags: 
editor: markdown
dateCreated: 2022-04-27T21:31:59.664Z
---

In order for Duo to use LDAPS (LDAP over SSL) authentication to communicate with Active Directory, you must already have a valid SSL certificate in use on your domain controller(s). If you need to set up a new SSL certificate for use with LDAPS, you can use the instructions in this Microsoft article:  [How to enable LDAP over SSL with a third-party certification authority](https://docs.microsoft.com/en-us/troubleshoot/windows-server/identity/enable-ldap-over-ssl-3rd-certification-authority).  

To export an issuing certificate chain from your certificate store to use with LDAPS authentication, use the following process.

On an Active Directory domain controller running on Windows Server 2012, open Start > Run > **certlm.msc **and skip ahead to step 7. On older Windows Server versions, open an administrative command prompt, type **mmc** to run Microsoft Management Console, and continue to step 2.

Click **File** **> Add/Remove Snap-in...**.

Select **Certificates **and click **Add > **to add the Certificate Manager snap-in.

Select **Computer account **and click **Next >**.

Make sure **Local computer **is selected and click **Finish**.

Click **OK**.

Navigate to **Certificates (Local Computer) > Personal > Certificates**.

Right-click the SSL certificate and click **Open**.

The  [acert.exe tool](https://help.duo.com/s/article/4207)  can be used to identify the SSL certificate that is being used for LDAPS authentication on your domain controller.

The SSL certificate will list **Server authentication** under the **Intended Purposes** column in the Certificate Manager

Go to **Certification Path **and select the top certificate.

Click **View Certificate**.  
 

![](https://help.duo.com/servlet/rtaImage?eid=ka04u000000cZXf&feoid=00N700000039b6e&refid=0EM70000000VZOi)

Go to the **Details** tab and select **Copy to File**.  
 

![](https://help.duo.com/servlet/rtaImage?eid=ka04u000000cZXf&feoid=00N700000039b6e&refid=0EM70000000VZOn)

In the Certificate Export Wizard, click **Next**.

Select **Base-64 encoded X.509 (.CER) **and click **Next**.  
 

![](https://help.duo.com/servlet/rtaImage?eid=ka04u000000cZXf&feoid=00N700000039b6e&refid=0EM70000000VZOx)

Click **Browse **to enter a name for your exported certificate save it in a specific directory. Click **Save **then click **Next >**.

Click **Finish** to export your certificate to the desired directory. Click **OK **after the export completes.  
 

![](https://help.duo.com/servlet/rtaImage?eid=ka04u000000cZXf&feoid=00N700000039b6e&refid=0EM70000000VZPM)

If there are intermediate issuing certificates below the root certificate, then repeat steps 1-15 for each of those certificates. Before continuing to the next step, ensure that you have a certificate file for each issuing certificate (root and intermediate). Do not export the Active Directory server certificate (leaf) itself.

Locate your exported certificates and open them with Notepad or Notepad++.

If there are both root and intermediate certificates, append the content of all the certificates into one certificate file with the intermediate certificates at the top, then root certificate at the bottom (i.e. in reverse of the issuing order). An example of the order for a root and 2 intermediate certificates:

[Intermediate certificate 2 - issued by Intermediate certificate 1]

[Intermediate certificate 1 - issued by Root certificate]

[Root certificate]

There should now be a certificate file with the entire issuing certificate chain. This file will allow Duo to trust the certificate chain that issued the SSL certificate used by Active Directory for LDAPS authentication.