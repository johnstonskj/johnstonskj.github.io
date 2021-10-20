---
title: Public PGP Keys
layout: page
---

This is my public PGP Key. To make use of it, you will want to have [GnuPG](http://gnupg.org/) installed. See the related links at the bottom right of this page for more information on Public key encryption.


[![PGP key](https://keys.openpgp.org/vks/v1/by-fingerprint/644CF77829C7C3BB5B868DC896A0664B9D482667)](https://img.shields.io/badge/pgp-96A0664B9D482667-blue) From October 2021 to present.

[![PGP key](https://keys.openpgp.org/vks/v1/by-fingerprint/49CC458D0281805F54A1B65A565B73DB6CC7F708)](https://img.shields.io/badge/pgp-565B73DB6CC7F708-inactive) From March 2019 to October 2021.

This key is used for signing and/or encrypting documents, like email messages or ordinary files. I only have one key at the moment.

## Using My Key(s)

This key is used for signing and/or encrypting documents, like email messages or ordinary files. 

You can download my [current PGP key](/assets/keys/644CF77829C7C3BB5B868DC896A0664B9D482667.asc) or from the key server [https://keys.openpgp.org/](https://https://keys.openpgp.org/).

If you do not have access to a keyserver, you can:

Firstly, download the [PGP public key file](/assets/keys/644CF77829C7C3BB5B868DC896A0664B9D482667.asc). Once you have a file with my key in it, perform the following command:

```bash
$ gpg --import /assets/keys/644CF77829C7C3BB5B868DC896A0664B9D482667.asc
```

Please feel free to sign my key, if you feel that you’ve verified my identity and that the key is mine. But don’t sign it with an exportable signature.



## Why Use PGP?

PGP (Pretty Good Privacy) allows me to digitally sign, or even encrypt, emails or files. Signed emails or files means that you can be sure it is I who sent them. Encrypted emails and files means that only someone using the proper PGP key can decrypt them.

Only someone who can access my computer and then unlock my PGP keyring can sign anything as me. But given that only I know the password for my PGP keyring, it is rather unlikely that my signature be forged.

I use OpenPGP. Visit the [GnuPG website](http://www.gnupg.org/) for more.
