---
title: Public PGP Keys
layout: page
---

The following is the public key for my public identity via [Gmail](https://www.google.com/gmail/about/). To make use of it, you will want to have [GnuPG](http://gnupg.org/) installed. 

[![PGP key](https://img.shields.io/badge/pgp-96A0664B9D482667-blue)](https://keys.openpgp.org/vks/v1/by-fingerprint/644CF77829C7C3BB5B868DC896A0664B9D482667) From October 2021 to present.

[![PGP key](https://img.shields.io/badge/pgp-565B73DB6CC7F708-inactive)](https://keys.openpgp.org/vks/v1/by-fingerprint/49CC458D0281805F54A1B65A565B73DB6CC7F708) From March 2019 to October 2021.

This key is used for signing and/or encrypting documents, like email messages or ordinary files. It is also used with [GitHub](https://github.com//) to sign code commits for some projects.

## Using My Key

You can download my current PGP key from the key server [keys.openpgp.org](https://keys.openpgp.org/vks/v1/by-fingerprint/644CF77829C7C3BB5B868DC896A0664B9D482667). If you do not have access to a keyserver, you can download the [PGP public key file](/assets/keys/644CF77829C7C3BB5B868DC896A0664B9D482667.asc) from this site.

Once you have the key file perform the following command:

```bash
$ gpg --import /assets/keys/644CF77829C7C3BB5B868DC896A0664B9D482667.asc
```

Please feel free to sign my key, if you feel that you’ve verified my identity and that the key is mine. But don’t sign it with an exportable signature.

## Why Use PGP?

PGP (Pretty Good Privacy) allows me to digitally sign, or even encrypt, emails or files. Signed emails or files means that you can be sure it is I who sent them. Encrypted emails and files means that only someone using the proper PGP key can decrypt them.

Only someone who can access my computer and then unlock my PGP keyring can sign anything as me. But given that only I know the password for my PGP keyring, it is rather unlikely that my signature be forged.

I use [OpenPGP](https://www.openpgp.org/) via [GnuPG](http://www.gnupg.org/). On macos I use [GPG Suite](https://gpgtools.org/) via [Homebrew](https://brew.sh/) (packages `gpg`, `gpg-suite-no-mail`, `gpg-suite-pinentry`). 

## Other Identities

For my personal identity I use [ProtonMail](https://protonmail.com/) (as well as the excellent [ProtonVPN](https://protonvpn.com/)).

[![PGP
key](https://img.shields.io/badge/pgp-5337DCA90E415DEB-blue)](https://keys.openpgp.org/vks/v1/by-fingerprint/FB67DDD277A98138222B014A5337DCA90E415DEB)

The following is my key for work.

[![PGP
key](https://img.shields.io/badge/pgp-B2DB6FC33732C4B8-blue)](https://keys.openpgp.org/vks/v1/by-fingerprint/30773E50D1FCC91F79B6F392B2DB6FC33732C4B8)

