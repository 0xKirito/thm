# THM

## Hashing - Crypto 101

### room/hashingcrypto101

Link: [hashingcrypto101](https://tryhackme.com/room/hashingcrypto101)

Note: This walkthrough is written with Kali Linux in mind. If you don't have any of the tools used here, you can download/install them manually.

### Tools We Will Use

#### hashid - Hash Identifier - Kali Linux Tools

|             **Parameter**             |               **Description**               |
| :-----------------------------------: | :-----------------------------------------: |
|      `hashid 'paste-hash-here'`       |       paste hash in quotes to analyze       |
|         `-e` or `--extended`          | list all hash algorithms & salted passwords |
|           `-m` or `--mode`            |       show **hashcat mode** in output       |
|           `-j` or `--john`            |   show **JohnTheRipper format** in output   |
| `-o file.txt` or `--outfile file.txt` |            write output to file             |
|               `--help`                |         show help message and exit          |
|              `--version`              |        show version number and exit         |

#### Hash Identifier Online Tools

- [Online Hash Crack - Hash Identifier](https://www.onlinehashcrack.com/hash-identification.php)
- [CyberChef](https://gchq.github.io/CyberChef/) - Paste hash & import **Analyse Hash** in **Recipe**.
- [TunnelsUP - Hash Analyzer](https://www.tunnelsup.com/hash-analyzer/)
- [Hashes - Hash Identifier](https://hashes.com/en/tools/hash_identifier)

#### Hash Cracking Online Tools

- [CyberChef](https://gchq.github.io/CyberChef/)
- [CrackStation](https://crackstation.net/)
- [Universal Encoding Tool - UnEnc](https://www.unenc.com/)
- [Hashes - Decrypt](https://hashes.com/en/decrypt/hash)

### Hashing - Crypto 101

Read the words, and understand the meanings! Is base64 encryption or encoding?

- encoding

What is the output size in bytes of the MD5 hash function?

- The hash size for the MD5 is 128 bits (8 bits = 1 byte).
- 16 bytes

Can you avoid hash collisions?

- Nay
- [Provably secure hash functions](https://en.wikipedia.org/wiki/Security_of_cryptographic_hash_functions#Provably_secure_hash_functions)

If you have an 8 bit hash output, how many possible hashes are there?

- The number of possibilities would be 2^(X) where X is the number of bits in the hash.
- MD5 outputs 128 bit hash value. So MD5 has 2^128 possible hashes.
- 8 bit hash value will have 2^8 possible hashes.

Crack the hash `d0199f51d2728db6011945145a1b607a` using the rainbow table manually.

- Visit [CrackStation](https://crackstation.net/) and paste the hash. You will get the password.

Crack the hash `5b31f93c09ad1d065c0491b764d04933` using online tools.

- Visit [hashes.com](https://hashes.com/en/decrypt/hash) and paste the hash. You will get the password.

Should you encrypt passwords?

- Nay (they should be hashed with a salt to make it really really hard to reverse them into plaintext passwords)

How many rounds does `sha512crypt ($6$)` use by default?

- 5000

What's the hashcat example hash (from the website) for Citrix Netscaler hashes?

- Visit [HashCat - Hashes Examples](https://hashcat.net/wiki/doku.php?id=example_hashes) and search `Citrix Netscaler hashes`. Copy the hash used there for example and submit it.

How long is a Windows NTLM hash, in characters?

- Visit [HashCat - Hashes Examples](https://hashcat.net/wiki/doku.php?id=example_hashes) and find NTLM hash example. Check the length of the given example.

Time to get cracking!
I'll provide the hashes. Crack them. You can choose how. You'll need to use online tools, Hashcat, and/or John the Ripper. Remember the restrictions on online rainbow tables. Don't be afraid to use the hints. `rockyou.txt` wordlist or online tools should be enough to find all of these.

Crack this hash: `$2a$06$7yoU3Ng8dHTXphAg913cyO6Bjs3K5lBnwq5FJyA6d01pMSrddr1ZG`.

- I'll provide the steps for Kali Linux. If you don't have any of these tools, you can download/install them manually. And I'll be saving these hashes in a local `hash.txt` file by overwriting it with a new hash for each question.
- `hashid -m 'paste-hash-here'` (paste hash inside quotes to avoid any issues) => 3200 => Blowfish
- `-m` = returns hashcat mode number
- Or paste it in some online hash identifiers. You will find it is bcrypt/Blowfish. But `hashid` method is faster. If you used some website, you might need to find hashcat mode number for that format using the following command.
- `hashcat -h | grep bcrypt` => 3200 (hashcat hash mode)
- Now to finally crack it...
- `hashcat -m 3200 hash.txt /usr/share/wordlists/rockyou.txt`
- If you don't mind some lag for faster cracking speed, you can use `-w` flag.
- `hashcat -m 3200 -w 3 hash.txt /usr/share/wordlists/rockyou.txt`
- `-w 3` = high workload = faster cracking. Find more information in `hashcat -h`.
- Wait for hashcat to crack the password. Note that this password is all numeric.

Crack this hash: `9eb7ee7f551d2f0ac684981bd1f1e2fa4a37590199636753efe614d4db30e8e1`.

- Visit [CrackStation](https://crackstation.net/) to crack this one. It is type sha256.

Crack this hash: `$6$GQXVvW4EuM$ehD6jWiMsfNorxy5SINsgdlxmAEl3.yif0/c3NqzGLa0P.S7KRDYjycw5bnYkF5ZtB8wQy8KnskuWQS3Yr1wQ0`.

- `hashid -m 'paste-hash-here'` (paste hash inside quotes to avoid any issues with hash identification) => 1800 => sha512crypt
- `hashcat -m 1800 -w 3 hash.txt /usr/share/wordlists/rockyou.txt` (you can skip `-w 3` if you don't want any lag and can wait longer)
- Wait for hashcat to crack the password. Note that this password is all alphabets.

Bored of this yet? Crack this hash: `b6b0d451bbf6fed658659a9e7e5598fe`.

- Try online tools and they will give you the password (hash format MD5).

What's the SHA1 sum for the amd64 Kali 2019.4 ISO? `http://old.kali.org/kali-images/kali-2019.4/`?

- Visit [Kali 2019.4](http://old.kali.org/kali-images/kali-2019.4/). Click on `SHA1SUMS` file and copy the relevant hash and submit it.

What's the hashcat mode number for `HMAC-SHA512 (key = $pass)`?

- `hashcat -h | grep HMAC-SHA512` and check the results.
