# Kerberos Overview
- Kerberos is the authentication and authorization protocol of choice in Active Directory environments. It can be attacked and exploited in several ways to move laterally within a network, including:
    - AS-REP and AS-REQ Roasting
    - Kerberoasting
    - UnConstrained Delegation
    - Constrained Delegation
    - Resource Based Constrained Delegation
    - Golden Tickets
    - Silver Tickets
- The aim of this project is to guide a penetration tester through enumerating and exploiting these numerous flaws. As such, we here will be focused on the pragmatic utility of each aspect of the Kerberos protocol, insofar as utility means anything furthering our access.

## Kerberos
- The main components of kerberos are a network of computers that share a common authentication database, and a key distribution center. The database stores Tickets (more on those later) granted for authentication and authorization to various components of the network.
- Tickets are issued by the KDC(Key Distribution Center) which in AD environments is usually the Domain Controller. These tickets are used to either authenticate to a service or request authentication tickets.
- A TGT (ticket granting ticket) is a ticket issued by the KDC that can be used to gain further authorization tickets to services in the network. A TGT is used for authentication, NOT authorization. This request and the following response are called AS-REQ and AS-REP respectively.
- A TGS (ticket granting service) is a ticket obtained from the KDC that can be used to gain access to services within a network. The two components, request and response for this ticket are dubbed TGS-REQ and TGS-REP respectively.
- Hence, in the normal course of events a user obtains a TGT from the KDC, then presents the TGT to obtain a TGS(for a service the KDC knows the user is allowed access), following which the user presents the TGS to the service, which validates the request, and if all is in order, grants the user access.

## Bruteforcing
- This attack focuses on bruteforcing usernames and passwords through the KDC to obtain valid credentials for further exploitation.
### User Enumeration
- `kerbrute userenum -d DOMAIN.local --dc DC01.DOMAIN.local <path\to\usernames.txt>`
- The above command is used to determine registered sAMAccountNames.
### Password Bruteforce
- Once a set of valid usernames have been determined, we can use kerbrute again to attempt to guess their passwords.
- `kerbrute bruteuser  -d DOMAIN.local --dc DC01.DOMAIN.local <path\to\passwords.txt> <username>`
- NOTE: Bruteforcing passwords must be done responsibly and sparingly as we may lock out accounts. To avoid this, it is best practice to get the password policy for the domain and use that information to limit our bruteforce attempts.

## AS-REQ Roasting
- An attacker may assume a MITM position, and listen for AS-REQ requests, which contain a timestamp encrypted with the requesting users account password. We as attackers may then proceed to crack the password using a bruteforce attack. A tool like `Pcredz` can be used to obtain passwords from a wireshark capture.
- `Pcredz -f path\to\file.pcap`

## AS-REP Roasting
- Among the four stages of a kerberos authentication-authorization cycle, this method targets the second stage, i.e AS-REP.
- Users with Kerberos Pre-Authentication disabeld (for various reasons, some applications demand this setting be turned on) are the target of this attack. As a consequence of disabling this setting, we may request a TGT for the user and be granted one without the burden of authentication.
- To list AS-REProastable users we run the following:
- `GetNPUsers.py DOMAIN.local/ -dc-ip <dc-ip>` OR  `Get-ADUser -Filter {DoesNotRequirePreAuth -eq $True} -Properties DoesNotRequirePreAuth`
- To request a ticket we run the following:
- `GetNPUsers.py DOMAIN.local/ -dc-ip <dc-ip> -request`
- We may then crack the hash and obtain our password.

## Kerberoasting
- This attack focuses on accounts with servicePrincipleNames(SPNs) set. We as attackers request a TGS for that user using a valid TGT. If a user has a SPN set, and we have a valid TGT, we may request a TGS for the target, and crack the TGS data(encrypted with the targets password) offline.
- Listing all kerberoastable users: `Get-ADUser -Filter {ServicePrincipalName -ne "$null"} -Properties ServicePrincipalName`
- The following commands are used to get the TGS:
- `GetUserSPNs.py 'DOMAIN.local/user:password' -dc-ip <dc-ip> -outputfile ofile.txt`
- We may then crack the hash offline

## Delegation
- 
