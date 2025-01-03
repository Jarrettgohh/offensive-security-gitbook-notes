---
description: >-
  The Windows Security Account Manager (SAM) database stores user passwords in a
  hashed format: LM or NTLM hash
---

# Windows SAM database

### Storage location of the SAM database

According to ChatGPT:

* **Main storage**: `%systemroot%\system32\config\sam`

Eg. `C:\Windows\System32\config\`

* **Backup**: `%systemroot%\repair\sam._`

Eg. `C:\Window\repair\sam._`

### General format of values in a SAM database entry

`username:RID:LM_hash:NT_hash:::`

1. `username`: Username of the account
2. `RID`: Unique relative identifier value
3. `LM_hash`: LM hash value. It is an older and weaker hash used for authentication. It created by appling the DES (Data Encryption Standard) algorithm on a given password.
4. `NT_hash`: It is the more secure and widely used hash in modern Windows systems. It can be generated by using the MD4 hashing algorithm on the UTF-16 representation of a given password.

#### Example entry in the SAM database

`jarrett:xxx:xxxx:xxxx:::`

### How to retrieve the NTLM hash?

1. `hashdump` command in a Metasploit's meterpreter shell
