---
title: LDAP Configuration for Brocade Switches
description: 
published: true
date: 2023-01-22T16:08:38.524Z
tags: brocade, ldap, security, user
editor: markdown
dateCreated: 2022-02-17T18:07:08.670Z
---

# Objective
To configure Brocade SAN switches or directors to use LDAP authentication for role-based access.  At the end of this procedure, you will have a domain group that is mapped to the "admin" role for all logical fabrics on the Brocade switch.  If logging in with the LDAP account fails, the local user database will still be used for logging in.

# Procedure
 
1. In order to enable LDAP authentication, you must first ensure than DNS is configured 
    ```plaintext
    dnsconfig --add -domain <domain_name> -serverip1 <ip_address_1> -serverip2 <ip_address_2>
    ```
    Verify the DNS configuration
    ```plaintext
    dnsconfig --show

         Domain Name Server Configuration Information 
         ____________________________________________ 

         Domain Name            = esilabs.com
         Name Server IP Address = 172.21.16.11
         Name Server IP Address = 172.22.16.12
    ```

1. Define the LDAP servers on the switch using the **aaaconfig** command
	Add the primary LDAP server
    ```plaintext
      aaaconfig --add <exampleserver1> -conf ldap -d <exampledomain.com>
    ```
  	Add the secondary LDAP server (optional, recommended)
    ```plaintext
      aaaconfig --add <exampleserver2> -conf ldap -d <exampledomain.com>
    ```
    Verify the LDAP configuration
    ```plaintext
    aaaconfig --show
          RADIUS CONFIGURATIONS
          =====================
          RADIUS configuration does not exist.

          LDAP CONFIGURATIONS
          ===================

          Position                 : 1
          Server                   : 172.21.16.11
          Port                     : 389
          Domain                   : esilabs.com
          Timeout(s)               : 3

          Position                 : 2
          Server                   : 172.22.16.12
          Port                     : 389
          Domain                   : esilabs.com
          Timeout(s)               : 3

          TACACS+ CONFIGURATIONS
          =====================
          TACACS+ configuration does not exist.

          Primary AAA Service: Switch database
          Secondary AAA Service: None
          Log Primary Authentication Status: Yes
    ```

1. Determine the LDAP group(s) that will be defined with a role on the switch. 
		Example:  We will map the LDAP group called **"Domain Admins"** to the **admin** role on the switch.
   
1. Map the LDAP group with Brocade admin role

    ```plaintext
    ldapcfg --maprole "Domain Admins" admin
    ```
1. Add addributes to the defined role to define admin roles in virtual fabrics 1-128 as well enabling the chassis admin role.
    ```plaintext
    ldapcfg --mapattr "Domain Admins" -l "admin=1-128" -h 128 -c admin
    ```
1. Verify the ldap configuration

    ```plaintext
    ldapcfg --show
        	LDAP Role       |               Switch Role             |       Home VF         |       Chassis Role
				-------------------------------------------------------------------------------------------------------------
        	Domain Admins   |       admin=1-128                     |       128             |       admin
				-------------------------------------------------------------------------------------------------------------
    ```
1. Change the authentication method to first try LDAP.  If that fails for any reason, the local database will be used.

    ```plaintext
    aaaconfig --authspec "ldap;local"
    ```
1. While remaining logged into the session as local admin, open a new ssh session to the switch and test logging in with the LDAP account.  You should be able to just use the LDAP username (no need for user@domain.com)

1. After logging in with ldap account, verify role based access by issuing commands such as:
		
    ```plaintext
    ldapcfg --show
    chassisshow
    cfgshow

# Related Topics
[Brocade SAN Switch/Director Implementation Checklist](/san/brocade/implementation-checklist)
    