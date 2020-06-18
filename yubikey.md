# Yubikey

## Resources

  * [PGP and SSH keys on a Yubikey NEO](https://www.esev.com/blog/post/2015-01-pgp-ssh-key-on-yubikey-neo/)
  * [Yubikey on Archlinux](https://wiki.archlinux.org/index.php/YubiKey)
  * [Recommandations from NIST : SP 800-57 Part 1](http://csrc.nist.gov/publications/PubsSPs.html#800-57-part1)

> These article is just a more recent version of the Eric Severance's work.
> All credits to him !

## Generate the master key

```

$ mv .gnupg .gnupg.orig
$ ln -s /media/USB .gnupg

$ echo "cert-digest-algo SHA512" >> .gnupg/gpg.conf
$ echo "default-preference-list SHA512 SHA384 SHA256 SHA224 AES256 AES192 AES CAST5 ZLIB BZIP2 ZIP Uncompressed" >> .gnupg/gpg.conf

$ gpg --expert --full-gen-key 

Please select what kind of key you want:
   (1) RSA and RSA (default)
   (2) DSA and Elgamal
   (3) DSA (sign only)
   (4) RSA (sign only)
   (7) DSA (set your own capabilities)
   (8) RSA (set your own capabilities)
   (9) ECC and ECC
  (10) ECC (sign only)
  (11) ECC (set your own capabilities)
  (13) Existing key
  (14) Existing key from card
Your selection? 11

Possible actions for a ECDSA/EdDSA key: Sign Certify Authenticate 
Current allowed actions: Sign Certify 

   (S) Toggle the sign capability
   (A) Toggle the authenticate capability
   (Q) Finished

Your selection? s

Possible actions for a ECDSA/EdDSA key: Sign Certify Authenticate 
Current allowed actions: Certify 

   (S) Toggle the sign capability
   (A) Toggle the authenticate capability
   (Q) Finished

Your selection? q
Please select which elliptic curve you want:
   (1) Curve 25519
   (3) NIST P-256
   (4) NIST P-384
   (5) NIST P-521
   (6) Brainpool P-256
   (7) Brainpool P-384
   (8) Brainpool P-512
   (9) secp256k1
Your selection? 3
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 
Key does not expire at all
Is this correct? (y/N) y

GnuPG needs to construct a user ID to identify your key.

Real name: Julien Vinet
Email address: contact@julienvinet.dev
Comment: 
You selected this USER-ID:
    "Julien Vinet <contact@julienvinet.dev>"

Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? o
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
gpg: /home/jvinet/.gnupg/trustdb.gpg: trustdb created
gpg: key 8A3A9D1CE64EE5A8 marked as ultimately trusted
gpg: directory '/home/jvinet/.gnupg/openpgp-revocs.d' created
gpg: revocation certificate stored as '/home/jvinet/.gnupg/openpgp-revocs.d/B9D63910B20B8D1D354181398A3A9D1CE64EE5A8.rev'
public and secret key created and signed.

pub   nistp256 2020-06-17 [C]
      B9D63910B20B8D1D354181398A3A9D1CE64EE5A8
uid                      Julien Vinet <contact@julienvinet.dev>

```

## Generate the encryption subkey

```
$ gpg --edit-key E64EE5A8

gpg> addkey
Please select what kind of key you want:
   (3) DSA (sign only)
   (4) RSA (sign only)
   (5) Elgamal (encrypt only)
   (6) RSA (encrypt only)
  (14) Existing key from card
Your selection? 6
RSA keys may be between 1024 and 4096 bits long.
What keysize do you want? (2048) 2048
Requested keysize is 2048 bits
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 1y
Key expires at Thu 17 Jun 2021 11:16:08 PM CEST
Is this correct? (y/N) y
Really create? (y/N) y
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.

sec  nistp256/8A3A9D1CE64EE5A8
     created: 2020-06-17  expires: never       usage: C   
     trust: ultimate      validity: ultimate
ssb  rsa2048/E41B1D5D62E705D0
     created: 2020-06-17  expires: 2021-06-17  usage: E   
[ultimate] (1). Julien Vinet <contact@julienvinet.dev>

gpg> save
```

## Make a backup of secret keys

```
$ gpg --export-secret-keys E64EE5A8 > \
  /media/USB/8A3A9D1CE64EE5A8-2020-06-17-E41B1D5D62E705D0-secret.pgp
```

## Generate the signing and authentication subkeys

From Yubikey Series 5, ykpersonnalize is replaced by [yubikey-manager](https://developers.yubico.com/yubikey-manager/) (called ykman in CLI).
On Archlinux, you can install it with [AUR](https://www.archlinux.org/packages/?name=yubikey-manager) :
```
$ yay -S yubikey-manager
```
> **Note :** After you install the yubikey-manager (which can be called by ykman in CLI), you need to enable ``pcscd.service`` to get it running

With new yubikey (Series 5 and above) it's not necessary to activate any special mode (default mode: OTP+FIDO+CCID, meaning that currently the OTP, FIDO and CCID subsystem of the key are enabled.)

```
$ gpg --delete-secret-keys E64EE5A8
$ gpg --import < /media/USB/8A3A9D1CE64EE5A8-2020-06-17-E41B1D5D62E705D0-secret.pgp

$ gpg --edit-key E64EE5A8

gpg> addcardkey
Signature key ....: [none]
Encryption key....: [none]
Authentication key: [none]

Please select the type of key to generate:
   (1) Signature key
   (2) Encryption key
   (3) Authentication key
Your selection? 1

Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 1y
Key expires at Thu 17 Jun 2021 11:39:11 PM CEST
Is this correct? (y/N) y
Really create? (y/N) y

sec  nistp256/8A3A9D1CE64EE5A8
     created: 2020-06-17  expires: never       usage: C   
     trust: ultimate      validity: ultimate
ssb  rsa2048/E41B1D5D62E705D0
     created: 2020-06-17  expires: 2021-06-17  usage: E   
ssb  rsa2048/DF071DC45C644EC7
     created: 2020-06-17  expires: 2021-06-17  usage: S   
     card-no: 0006 12083527
[ultimate] (1). Julien Vinet <contact@julienvinet.dev>

gpg> addcardkey 
Signature key ....: DB57 BDF4 C2FE 919F 6E56  3011 DF07 1DC4 5C64 4EC7
Encryption key....: [none]
Authentication key: [none]

Please select the type of key to generate:
   (1) Signature key
   (2) Encryption key
   (3) Authentication key
Your selection? 3

Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 1y
Key expires at Thu 17 Jun 2021 11:39:55 PM CEST
Is this correct? (y/N) y
Really create? (y/N) y

sec  nistp256/8A3A9D1CE64EE5A8
     created: 2020-06-17  expires: never       usage: C   
     trust: ultimate      validity: ultimate
ssb  rsa2048/E41B1D5D62E705D0
     created: 2020-06-17  expires: 2021-06-17  usage: E   
ssb  rsa2048/DF071DC45C644EC7
     created: 2020-06-17  expires: 2021-06-17  usage: S   
     card-no: 0006 12083527
ssb  rsa2048/E069F0C0AB7077FD
     created: 2020-06-17  expires: 2021-06-17  usage: A   
     card-no: 0006 12083527
[ultimate] (1). Julien Vinet <contact@julienvinet.dev>

gpg> toggle

sec  nistp256/8A3A9D1CE64EE5A8
     created: 2020-06-17  expires: never       usage: C   
     trust: ultimate      validity: ultimate
ssb  rsa2048/E41B1D5D62E705D0
     created: 2020-06-17  expires: 2021-06-17  usage: E   
ssb  rsa2048/DF071DC45C644EC7
     created: 2020-06-17  expires: 2021-06-17  usage: S   
     card-no: 0006 12083527
ssb  rsa2048/E069F0C0AB7077FD
     created: 2020-06-17  expires: 2021-06-17  usage: A   
     card-no: 0006 12083527
[ultimate] (1). Julien Vinet <contact@julienvinet.dev>

gpg> key 1

sec  nistp256/8A3A9D1CE64EE5A8
     created: 2020-06-17  expires: never       usage: C   
     trust: ultimate      validity: ultimate
ssb* rsa2048/E41B1D5D62E705D0
     created: 2020-06-17  expires: 2021-06-17  usage: E   
ssb  rsa2048/DF071DC45C644EC7
     created: 2020-06-17  expires: 2021-06-17  usage: S   
     card-no: 0006 12083527
ssb  rsa2048/E069F0C0AB7077FD
     created: 2020-06-17  expires: 2021-06-17  usage: A   
     card-no: 0006 12083527
[ultimate] (1). Julien Vinet <contact@julienvinet.dev>

gpg> keytocard
Please select where to store the key:
   (2) Encryption key
Your selection? 2

sec  nistp256/8A3A9D1CE64EE5A8
     created: 2020-06-17  expires: never       usage: C   
     trust: ultimate      validity: ultimate
ssb* rsa2048/E41B1D5D62E705D0
     created: 2020-06-17  expires: 2021-06-17  usage: E   
ssb  rsa2048/DF071DC45C644EC7
     created: 2020-06-17  expires: 2021-06-17  usage: S   
     card-no: 0006 12083527
ssb  rsa2048/E069F0C0AB7077FD
     created: 2020-06-17  expires: 2021-06-17  usage: A   
     card-no: 0006 12083527
[ultimate] (1). Julien Vinet <contact@julienvinet.dev>

gpg> save
```

## Save and distribute the public OpenPGP key

```
$ gpg --edit-key E64EE5A8

gpg> keyserver
Enter your preferred keyserver URL: https://keybase.io/knacky/key.asc

sec  nistp256/8A3A9D1CE64EE5A8
     created: 2020-06-17  expires: never       usage: C   
     trust: ultimate      validity: ultimate
ssb  rsa2048/E41B1D5D62E705D0
     created: 2020-06-17  expires: 2021-06-17  usage: E   
     card-no: 0006 12083527
ssb  rsa2048/DF071DC45C644EC7
     created: 2020-06-17  expires: 2021-06-17  usage: S   
     card-no: 0006 12083527
ssb  rsa2048/E069F0C0AB7077FD
     created: 2020-06-17  expires: 2021-06-17  usage: A   
     card-no: 0006 12083527
[ultimate] (1). Julien Vinet <contact@julienvinet.dev>

gpg> showpref
[ultimate] (1). Julien Vinet <contact@julienvinet.dev>
     Cipher: AES256, AES192, AES, CAST5, 3DES
     Digest: SHA512, SHA384, SHA256, SHA224, SHA1
     Compression: ZLIB, BZIP2, ZIP, Uncompressed
     Features: MDC, Keyserver no-modify
     Preferred keyserver: https://keybase.io/knacky/key.asc

gpg> save

$ gpg --armor --export E64EE5A8 > E64EE5A8.asc
```

## Remove the master key and update the Yubikey

```
$ rm .gnupg

$ mv .gnupg.orig .gnupg

$ gpg --card-edit

Reader ...........: 1050:0407:X:0
Application ID ...: D2760001240103040006120835270000
Application type .: OpenPGP
Version ..........: 3.4
Manufacturer .....: Yubico
Serial number ....: 12083527
Name of cardholder: [not set]
Language prefs ...: [not set]
Salutation .......: 
URL of public key : [not set]
Login data .......: [not set]
Signature PIN ....: not forced
Key attributes ...: rsa2048 rsa2048 rsa2048
Max. PIN lengths .: 127 127 127
PIN retry counter : 3 0 3
Signature counter : 2
KDF setting ......: off
Signature key ....: DB57 BDF4 C2FE 919F 6E56  3011 DF07 1DC4 5C64 4EC7
      created ....: 2020-06-17 21:39:04
Encryption key....: 1559 7154 5AB8 0F00 0608  E464 E41B 1D5D 62E7 05D0
      created ....: 2020-06-17 21:15:47
Authentication key: 4B5A ECDA 7F53 BC34 185C  EB8D E069 F0C0 AB70 77FD
      created ....: 2020-06-17 21:39:52
General key info..: [none]

gpg/card> admin
Admin commands are allowed

gpg/card> passwd
gpg: OpenPGP card no. D2760001240103040006120835270000 detected

1 - change PIN
2 - unblock PIN
3 - change Admin PIN
4 - set the Reset Code
Q - quit

Your selection? 1
Error changing the PIN: Bad PIN

1 - change PIN
2 - unblock PIN
3 - change Admin PIN
4 - set the Reset Code
Q - quit

Your selection? 1
Error changing the PIN: Bad PIN

1 - change PIN
2 - unblock PIN
3 - change Admin PIN
4 - set the Reset Code
Q - quit

Your selection? 1
Error changing the PIN: Conditions of use not satisfied

1 - change PIN
2 - unblock PIN
3 - change Admin PIN
4 - set the Reset Code
Q - quit

Your selection? 1
PIN changed.

1 - change PIN
2 - unblock PIN
3 - change Admin PIN
4 - set the Reset Code
Q - quit

Your selection? 3
PIN changed.

1 - change PIN
2 - unblock PIN
3 - change Admin PIN
4 - set the Reset Code
Q - quit

Your selection? q

gpg/card> forcesig

gpg/card> url
URL to retrieve public key: https://keybase.io/knacky/key.asc

gpg/card> fetch

gpg/card> quit

$ gpg --card-status
Application ID ...: D2760001240103040006120835270000
Application type .: OpenPGP
Version ..........: 3.4
Manufacturer .....: Yubico
Serial number ....: 876543210
Name of cardholder: [not set]
Language prefs ...: [not set]
Salutation .......: 
URL of public key : https://keybase.io/knacky/key.asc
Login data .......: [not set]
Signature PIN ....: forced
Key attributes ...: rsa2048 rsa2048 rsa2048
Max. PIN lengths .: 127 127 127
PIN retry counter : 3 3 3
Signature counter : 2
KDF setting ......: off
Signature key ....: DB57 BDF4 C2FE 919F 6E56  3011 DF07 1DC4 5C64 4EC7
      created ....: 2020-06-17 21:39:04
Encryption key....: 1559 7154 5AB8 0F00 0608  E464 E41B 1D5D 62E7 05D0
      created ....: 2020-06-17 21:15:47
Authentication key: 4B5A ECDA 7F53 BC34 185C  EB8D E069 F0C0 AB70 77FD
      created ....: 2020-06-17 21:39:52
General key info..: sub  rsa2048/DF071DC45C644EC7 2020-06-17 Julien Vinet <contact@julienvinet.dev>
sec#  nistp256/8A3A9D1CE64EE5A8  created: 2020-06-17  expires: never     
ssb>  rsa2048/E41B1D5D62E705D0  created: 2020-06-17  expires: 2021-06-17
                                card-no: 0006 12083527
ssb>  rsa2048/DF071DC45C644EC7  created: 2020-06-17  expires: 2021-06-17
                                card-no: 0006 12083527
ssb>  rsa2048/E069F0C0AB7077FD  created: 2020-06-17  expires: 2021-06-17
                                card-no: 0006 12083527
```


## Known issues

### EstablishContextException: 'Failure to establish context: Service not available.'

For Archlinux user :
```
$ yay -S acsccid
$ sudo systemctl enable pcscd.service
$ sudo systemctl start pcscd.service 
$ sudo systemctl status pcscd.service
```

### Kill/Restart GPG Agent

```
gconf --kill gpg-agent 
```