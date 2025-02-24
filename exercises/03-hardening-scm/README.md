# Hardening Source Code Management (SCM)

## Exercise: Fork This Repository

Let's get started! The first step is forking this repository into your GitHub account. We're working with the repository for some exercises from now on.

## Exercise: Sign Your Commits with Gitsign

### Background

As ["Should You Sign Git Commits?"](https://dlorenc.medium.com/should-you-sign-git-commits-f068b07e1b1f) explains, suppose that your GitHub account is compromised, commit signing with PGP keys is not effective to identify whether a person who pushed a commit.

This is because what GitHub does is **just verifying a signature with one of public keys registerd for the GitHub account**, although attackers will have add permissions to add a public key as well as adding commits when they compromised the account. If one wants to verify a signed commit is really by the account owner and not by attackers, he/she need to retrive a public key from outside GitHub.

[Keybase](https://keybase.io/) might be one of fine solutions for distributing one's public keys for that purpose, but not so many developers are using that kind of services (_sigh_).

### Gitsign

**Here comes [gitsign](https://github.com/sigstore/gitsign)**. It enables us to sign commits upon Sigstore PKI infrastructure; instead of using your local PGP keys, we can sign commits with ephemeral keys and relate one's identity in a OIDC IdP with commits. In other words, given attackers compromised your GitHub account, one may still find out what kind of identity made commits **unless both your GitHub account and account in another identity provider are hacked at once**. Gitsign will be a helpful alternative to public key distribution outside from GitHub.

Let's get started by installing `gitsign` locally:

```sh
go install github.com/sigstore/gitsign@4bc492cc12d32998473bddd01c31f2a97b94fd1c
```

...and configure your local environment to work with `gitsign` as follows:

```sh
cd <path/to/repository/you/forked>

# configure `git` to use `gitsign` only for thi repository
git config --local commit.gpgsign true
git config --local tag.gpgsign true
git config --local gpg.x509.program gitsign
git config --local gpg.format x509
```

All set! Now you can make a signed commit as follows (**note that you'll run into the same problems as the aforementioned ones if you use GitHub to log in to `oauth2.sigstore.dev`**):

```sh
git commit --allow-empty --message="Signed commit"
```

After making a commit, one can verify the commit with `git verify-commit` command:

```sh
$ git verify-commit HEAD -v

tree 820248303ed658ba3b0fe4736d80b3f91f013652
parent c778ddac18a8255d3e7c5c84bf4753c3d3671de5
author Takashi Yoneuchi <takashi.yoneuchi@shift-js.info> 1660164723 +0900
committer Takashi Yoneuchi <takashi.yoneuchi@shift-js.info> 1660164723 +0900

Signed commit
tlog index: 3165300
gitsign: Signature made using certificate ID 0x2fb8dfc6064f51280b6be70a97b4ea6be6143530 | CN=sigstore-intermediate,O=sigstore.dev
gitsign: Good signature from [t.yoneuchi@flatt.tech]
Validated Git signature: true
Validated Rekor entry: true
```

`tlog index` points to a Rekor record. The record will look like:

```sh
$ rekor-cli get --log-index 3165300 --format json | jq -r ".Body.HashedRekordObj.signature.publicKey.content" | base64 -d | openssl x509 -text

Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            23:47:4f:65:08:58:6d:39:98:35:88:87:97:51:c4:af:5b:6c:54:7d
        Signature Algorithm: ecdsa-with-SHA384
        Issuer: O = sigstore.dev, CN = sigstore-intermediate
        Validity
            Not Before: Aug 10 20:52:29 2022 GMT
            Not After : Aug 10 21:02:29 2022 GMT
        Subject:
        Subject Public Key Info:
            Public Key Algorithm: id-ecPublicKey
                Public-Key: (256 bit)
                pub:
                    04:d9:ac:1d:de:05:03:94:d0:23:ac:10:f5:a2:f9:
                    f1:22:17:27:fe:91:8a:2f:43:79:a6:1d:9c:ba:78:
                    64:f6:a2:ef:91:53:d5:22:6b:72:dd:24:28:89:6c:
                    14:6c:ec:03:81:e6:d9:54:00:8e:97:64:13:1e:c2:
                    96:87:ae:8e:34
                ASN1 OID: prime256v1
                NIST CURVE: P-256
        X509v3 extensions:
            X509v3 Key Usage: critical
                Digital Signature
            X509v3 Extended Key Usage:
                Code Signing
            X509v3 Subject Key Identifier:
                18:92:BD:FF:1B:8A:EE:CA:7D:80:15:1B:A1:D6:D0:BB:F8:E2:E3:A3
            X509v3 Authority Key Identifier:
                DF:D3:E9:CF:56:24:11:96:F9:A8:D8:E9:28:55:A2:C6:2E:18:64:3F
            X509v3 Subject Alternative Name: critical
                email:t.yoneuchi@flatt.tech
            1.3.6.1.4.1.57264.1.1:
                https://accounts.google.com
            CT Precertificate SCTs:
                Signed Certificate Timestamp:
                    Version   : v1 (0x0)
                    Log ID    : 08:60:92:F0:28:52:FF:68:45:D1:D1:6B:27:84:9C:45:
                                67:18:AC:16:3D:C3:38:D2:6D:E6:BC:22:06:36:6F:72
                    Timestamp : Aug 10 20:52:30.004 2022 GMT
                    Extensions: none
                    Signature : ecdsa-with-SHA256
                                30:45:02:20:14:0B:0E:64:A1:1B:17:84:49:2A:2B:82:
                                46:DC:D6:CA:0A:DC:13:4C:9A:41:00:BA:83:A0:B2:E2:
                                93:F0:B0:99:02:21:00:A5:04:C1:FD:8A:3D:FC:7D:31:
                                E0:F8:E2:08:52:88:54:D6:01:74:0D:AD:88:03:FD:51:
                                CF:73:13:AF:D4:E0:57
    Signature Algorithm: ecdsa-with-SHA384
    Signature Value:
        30:65:02:30:56:4f:76:6c:8a:1b:14:61:08:8c:a8:a9:2c:26:
        54:c4:d5:df:94:1b:c9:f0:27:e7:3d:e6:f1:a5:76:00:94:88:
        32:92:a8:53:4a:4d:e6:8a:e8:09:af:61:58:1e:28:6d:02:31:
        00:ef:de:38:26:23:90:8f:9d:1a:d0:27:02:e3:5d:9c:fa:da:
        72:c9:70:2b:c5:96:dc:ff:09:d1:d7:55:38:a4:ed:60:3e:4d:
        df:1b:34:3a:66:30:08:68:45:7e:28:56:96
-----BEGIN CERTIFICATE-----
MIICoTCCAiegAwIBAgIUI0dPZQhYbTmYNYiHl1HEr1tsVH0wCgYIKoZIzj0EAwMw
NzEVMBMGA1UEChMMc2lnc3RvcmUuZGV2MR4wHAYDVQQDExVzaWdzdG9yZS1pbnRl
cm1lZGlhdGUwHhcNMjIwODEwMjA1MjI5WhcNMjIwODEwMjEwMjI5WjAAMFkwEwYH
KoZIzj0CAQYIKoZIzj0DAQcDQgAE2awd3gUDlNAjrBD1ovnxIhcn/pGKL0N5ph2c
unhk9qLvkVPVImty3SQoiWwUbOwDgebZVACOl2QTHsKWh66ONKOCAUYwggFCMA4G
A1UdDwEB/wQEAwIHgDATBgNVHSUEDDAKBggrBgEFBQcDAzAdBgNVHQ4EFgQUGJK9
/xuK7sp9gBUbodbQu/ji46MwHwYDVR0jBBgwFoAU39Ppz1YkEZb5qNjpKFWixi4Y
ZD8wIwYDVR0RAQH/BBkwF4EVdC55b25ldWNoaUBmbGF0dC50ZWNoMCkGCisGAQQB
g78wAQEEG2h0dHBzOi8vYWNjb3VudHMuZ29vZ2xlLmNvbTCBigYKKwYBBAHWeQIE
AgR8BHoAeAB2AAhgkvAoUv9oRdHRayeEnEVnGKwWPcM40m3mvCIGNm9yAAABgomH
urQAAAQDAEcwRQIgFAsOZKEbF4RJKiuCRtzWygrcE0yaQQC6g6Cy4pPwsJkCIQCl
BMH9ij38fTHg+OIIUohU1gF0Da2IA/1Rz3MTr9TgVzAKBggqhkjOPQQDAwNoADBl
AjBWT3ZsihsUYQiMqKksJlTE1d+UG8nwJ+c95vGldgCUiDKSqFNKTeaK6AmvYVge
KG0CMQDv3jgmI5CPnRrQJwLjXZz62nLJcCvFltz/CdHXVTik7WA+Td8bNDpmMAho
RX4oVpY=
-----END CERTIFICATE-----
```

## Exercise: Hardening Your GitHub Repository

Visit ["開発者のための GitHub Organization の安全な運用と継続的なモニタリング"](https://speakerdeck.com/flatt_security/kai-fa-zhe-falsetamefalse-github-organization-falsean-quan-nayun-yong-to-ji-sok-de-namonitaringu) and try to harden your GitHub repository.

## Hardening GitLab

> **Info**
> Coming soon!

## Hardening Bitbucket

> **Info**
> Coming soon!
