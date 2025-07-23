![Eove's logo](http://eove.fr/cms/wp-content/uploads/logo-1.png)

# IN69 Cérémonie de vérifications récurrentes de la PKI d'entreprise










Révision @ORCA@rev@


| Edité par                             | Vérifié par                           | Apprové par                     |
| ------------------------------------- | ------------------------------------- | ------------------------------- |
| L. Ains                               | X. Detant                             |                                 |
| Date 12/05/2025                       | Date 12/05/2025                       |                                 |
| Signature digitale en fin de document | Signature digitale en fin de document | Visa                            |

Document généré depuis les sources dans le dépôt git [@ORCA@gitremote@/tree/main/vault/docs](@ORCA@gitremote@/tree/main/vault/docs) au commit :  
`@ORCA@commit@`

## Protocole de vérification

> [!Tip]  
> L'authenticité du contenu de ce document IN69 (au format HTML), doit être vérifié à l'aide de signature cryptographiques avant d'entamer son déroulement.
> * Retrouver les identités publiques gpg des signataires (éditeur et vérificateur). Pour cela, se placer au commit signalé dans l'entête :
> `git checkout @ORCA@commit@`
> * À ce commit, les clés publiques se trouvent dans le répertoire [`src/workflow_signatory_keys/`](@ORCA@gitremote@/tree/main/src/workflow_signatory_keys).
> * Vérifier que ces clés ont bien été insérées par un **commit signé par leur propriétaire**.
> * Utiliser un nouveau trousseau gpg vide (toutes les commandes ci-dessous seront ensuite exécutées avec la variable d'environnement) :  
> `export TMP_GPG_HOME=$(mktemp -d)`
> * Importer toutes les clés publiques dans ce nouveau trousseau :  
> `gpg --home="$TMP_GPG_HOME" --import /path/to/src/workflow_signatory_keys/*`
> * Marquer toutes les clés comme de confiance ultime :  
> `gpg --home="$TMP_GPG_HOME" --list-keys --keyid-format LONG --with-colons | sed -n -e '/^pub/{n;p}' | sed -n -E 's/^fpr:([^:]*:){8}([^:]*).*$/\2:6:/p' | gpg --home="$TMP_GPG_HOME"  --import-ownertrust`
> * Extraire les signatures de la fin du document html :  
> `sed -e '1,/^<!-- @GPG@SIGNATURES@ --><pre>$/ d' /chemin/vers/IN69_signed.html > /tmp/IN69.sig`
> * Extraire la vestion html d'origine du document (sans signatures) :  
> `sed -e '/^<!-- @GPG@SIGNATURES@ --><pre>$/q' /chemin/vers/IN69_signed.html > /tmp/IN69_without_signatures.html`
> * Vérifier toutes les signatures :  
> `gpg --home="$TMP_GPG_HOME" --verify /tmp/IN69.sig /tmp/IN69_without_signatures.html && echo "All signatures verified"`
> * La validité des signatures est confirmée par l'affichage de la ligne `Good signature from xxxx` **autant de fois que de signataires du document** et qu'une dernière ligne `All signatures verified` s'affiche.

## Introduction

Ce document décrit le déroulement de la cérémonie de vérifications récurrentes de la PKI d'Eove (chaîne de confiance basée sur des certificats digitaux).
Ce processus implique l'ouverture de la PKI offline si des opérations doivent être effectuées sur cette dernière et dépend donc de l'exécution de la fiche IN65.

## Historique des révisions

| Révision |    Date    |   Auteur       | Description |
| -------- | :--------: | :------------: | ----------: |
| A        | 12/05/2025 | Lionel Ains    | Création    |

{{#include ../common/glossary.md}}

## Protocole de vérification

### Objectif

Le protocole détaillé ci-dessous indique les étapes à suivre pour vérifier que la PKI Eove est toujours utilisable de façon pérenne.
Cela inclut des vérifications sur la CA online et la CA offline.

Le reste de ce document est rédigé en anglais.

## Overview

### Architecture of the vault system

{{#include ../../architecture.md}}

### Note on scripts

{{#include ../../note-on-scripts.md}}

## Prerequisites

In order to execute the current document, you will need:
* An initialised offline vault
* An initialised and running online vault
* To fulfill all the prerequisites to run an IN65 ceremony

## Recurrent checks

On a regular basis, the whole PKI usability should be assessed, this means that **all** the following checks should pass or be fixed by an action described in each chapter.

Checks (and fixes) below should be performed first on the preprod environment, then on prod.
If fixes are needed, this means running an IN65 on preprod, and then an IN65 on prod.

> [!Warning]  
> In order to use scripts on the offline vault, they should be copied and adapted from the directory `templates/` to `src/scripts/`, then added to git and only then, the build of the live bootable media will make these scripts available from the command line on the offline terminal.

### Yubikeys certificates that have expired should be renewed

Yubikeys' certificates last for at most 8 years.

Every year, we are expecting a minimum of 1 share to be renewed.

> [!Warning]  
> An absolute maximum of no more than the total number of shares - the quorum number be renewed during the same share rotation.
> This maximum is set to prevent from locking us out of unsealing vaults during the share rotation.
> If you plan to rotate this maximum number of shares, you should make sure that the non-renewed key are still usable before regenerating the keys on the hardware tokens.

Renewal for all currently in-use Yubikeys should be spread over the full validity period of Yubikeys certificates.
Taking 8 years as the default validity, if we have 11 valid shares (Yubikeys), then we should have a schedule of 2 Yubikeys regeneration per year for the first 3 years and 1 Yubikey for the next 5 years.
This is especially important when initially generating certificates on Yubikeys. Many Yubikeys will not be initially valid for the whole 8 years.

#### Test

Yubikeys certificates expire on 30/12 of the year mentioned in their public key filename (as registered in the folder `share_holders/`)
If a Yubikey certificate is expired when performing this check, then it should be renewed.

The 📢`organiser` should perform this check and ask the relevant share holders to renew their Yubikey's GPG key pair.

#### Fix

Yubikey's GPG public key should be updated and commited by their owner in the relevant folder `share_holders/`.  
The filename storing the Yubikey's public key should follow the template:  
`2024_firstname.lastname@email.com_yubikey-62413455.pub`  
Where:
* `2024` is the expiry year of the Yubikey (it will expire on 30/12/2024), you can double-check the expiry using `gpg /path/to/file.pub`
* `firstname.lastname@email.com` is the email address of the owner of this Yubikey
* `62413455` is the serial number of the Yubikey (as displayed, for example using `gpg --card-status`)

Once all relevant GPG keys have been renewed and their public key commited to the repository, the following script should be run in an IN65 workflow for the offline vault:  
`templates/unauthenticated/rotate-seal-shares.sh`

An unseal share rotation should be run also on the online vault.

### Every share holder can use the unseal share they own

And every share holder with a Yubikey should have a GPG public key registered in the folder `share_holders/`

Every share holder runs the following tests with their Yubikey inserted

#### Test

```bash
gpg --card-status
```

This should return the serial number of the Yubikey.

> [!Tip]  
> If you have an error message `gpg: selecting card failed: No such device` and/or `gpg: OpenPGP card not available: No such device`, try to run:\
> `pkill gpg-agent`

If the Yubikey is communicating properly, we will extract its key ID:
```bash
export GPG_HW_TOKEN_KEY_ID=$(gpg --card-status | sed -n -E -e 's/^[^:]*sign[^:]*:[[:blank:]]*((:?[[:xdigit:]]{4}[[:blank:]]*){10})/\1/pi') && echo "$GPG_HW_TOKEN_KEY_ID"
```

Now let's test that the yubikey actually works (can decrypt data when it's the targeted recipient, and matches a correctly exported public GPG key):
```bash
export TMP_GPG_HOME=$(mktemp -d)
cp ~/.gnupg/*.conf "$TMP_GPG_HOME"/
gpg --home="$TMP_GPG_HOME" --import /path/to/src/share_holders_keys/env/*
gpg --home="$TMP_GPG_HOME" --list-keys --keyid-format LONG --with-colons | sed -n -e '/^pub/{n;p}' | sed -n -E 's/^fpr:([^:]*:){8}([^:]*).*$/\2:6:/p' | gpg --home="$TMP_GPG_HOME" --import-ownertrust
echo "It works" | gpg --home="$TMP_GPG_HOME" -e -r "$GPG_HW_TOKEN_KEY_ID" | gpg -d
```

If the message `It works` is displayed, then your Yubikey is usable. This test is a PASS.  
If not, the Yubikey owner should notify the organiser.  
If at least one share holder fails to use their Yubikey even after troubleshooting, the organiser will organise the fix below.

#### Fix

The GPG keypair should be regenerated on the Yubikey of concern, keeping in mind that the new certificate generated on that Yubikey should still have the same expiry date as initially (to keep a correct spread of expiry dates).

The process is detailed [here](@ORCA@gitremote@/blob/main/docs/yubikeys.md#generating-a-new-opengpg-key).

Once a new keypair has been set up on the Yubikey, extract your public key and update the env-specific directory located under folder `src/share_holders_keys/` in this repository.

Finally, force an unseal share rotation by executing the following script. It should be run in an IN65 workflow for the offline vault:  
`templates/unauthenticated/rotate-seal-shares.sh`

An unseal share rotation should be run also on the online vault.

### All target shared holders should have a hardware token attributed

Every shareholder should have a Yubikey, and these Yubikey's GPG keys' details should match their owner's name and e-mail address at Eove.

Every share holder runs the following tests with their Yubikey inserted

#### Test

```bash
gpg --card-status
```

The details of your Yubikey will be displayed, including the serial number, that should match the [Share holders spreadsheet, both in preprod and prod folder](https://drive.google.com/drive/folders/1NJg2EfFr1zx0L8r91-SeJX-d9baYWthM).

The following command will get the username and email address associated with your Yubikey.
```bash
gpg --card-status  | sed -n -e '/^General key info/,//p' | sed -n -e 's@^.*pub[[:blank:]][[:blank:]]*@@p'
```

The username should match your identity, the e-mail address should be yours (@eove.fr).

If this is the case, this test is a PASS.  
If not, the Yubikey owner should notify the organiser.  
If at least one share holder fails to use their Yubikey even after troubleshooting, the organiser will organise the fix below.

#### Fix

The GPG keypair should be regenerated on the Yubikey of concern, keeping in mind that the new certificate generated on that Yubikey should still have the same expiry date as initially (to keep a correct spread of expiry dates).

The process is detailed [here](@ORCA@gitremote@/blob/main/docs/yubikeys.md#generating-a-new-opengpg-key).

Once a new keypair has been setup on the Yubikey, extract your public key and update the env-specific directory located under folder `src/share_holders_keys/` in this repository.

Finally, force an unseal share rotation by executing the following script. It should be run in an IN65 workflow for the offline vault:  
`templates/unauthenticated/rotate-seal-shares.sh`

An unseal share rotation should be run also on the online vault.

### Valid certificates can be issued by the online CA during the next 18 months to come

The online CA that signs device certificates should be able to perform valid signatures until the next recurrent check (plus some margin), that is to say, for the next 18 months.

#### Test

Take the current date now, it will be called *D<sub>now</sub>*. Add 18 months to *D<sub>now</sub>*, then add again the duration of devices certificates. The result is a date *D<sub>min</sub>*.

In order to get the expiry date of the online CA currently in use to sign devices, run the following command against the **online vault** after being logged in to the vault using an account with **admin** privileges:
```bash
vault read -format=json devices_pki/issuer/default/json | jq -r '.data.certificate' | openssl x509 -noout -text
```

The start of validity date will be called *D<sub>start</sub>*, it can be found in the `Validity`'s `Not Before` attribute displayed by the command above.
The expiry date will be called *D<sub>expiry</sub>*, it can be found in the `Validity`'s `Not After` attribute displayed by the command above.

The following statement should be true:
*D<sub>start</sub>*<*D<sub>now</sub>* **and** *D<sub>expiry</sub>*>*D<sub>min</sub>*.
If this is the case, this test is a PASS.  
If not, a new online CA should be generated, the fix below should be applied.

#### Fix

A new device signing online CA should be created, signed by the offline CA and setup to replace the current online CA in use.

> [!Warning]  
> The previous device signing online CAs should however **not be revoked**.

Before starting to generate a new device signing CA certificate, connect to the online vault and write down the current **default** issuer ID (linked to the current certificate):
```bash
vault read -format=json devices_pki/issuer/default/json | jq -r '.data.issuer_id'
```

> [!Note]  
> This obviously only applies to renewal of the device signing CA's certificate, not to the initial generation of the very first device signing only CA.

Set your vault environment:
```bash
export TARGET_VAULT=preprod
```

or 

```bash
export TARGET_VAULT=prod
```

> [!Note]  
> External CA generation should not be allowed on the online CA and would probably thus fail due to access rights.  
> This is on purpose, to prevent the private key of the generated CA from being exposed.  
> We are thus using internal generation.

Generate a new **internal** CA CSR on the online vault, by running the following script against the **online vault**:
```bash
./scripts/maintenance/create-devices-csr.sh
```

Copy the template `templates/authenticated/sign-csr.sh` into `src/scripts/authenticated` and embed the CSR content inside that copied version of the script.

This CSR content (PEM-formatted file) will be reviewed during the verification phase of the IN65 workflow.

Finally, on the offline vault, organise an IN65 workflow to execute the updated sign-csr script. The organiser should also direct all team members to the instructions below. Indeed, these provide step-by-step instructions on how to make sure the CSR comes from the online vault.

> [!Note]  
> Because the online vault has been configured to only generate internal CSR (the associated private key never leaves the vault), we are sure that if the CSR is coming from the online vault, then only that vault can use it and is seen as trusted by the offline vault.

When the CSR is generated, it is uniquely identified by its PKID property (called `subject_key_id` in hashicorp vault's terminology).

This PKID value should be identical between:
* the PEM-formatted CSR (that will be inlined inside the `sign-csr` script).
* the online vault that emitted the CSR.

You can copy the following lines in a file and run the script to automatically do that check.
```bash
#!/usr/bin/env bash

set -e

CSR="$1"

if [ -z $CSR ]
then
    echo "Usage $0 /path/to/file.csr"
    exit -1
fi

# Get the PKID of the given CSR
SUBJECT_PUBLIC_KEY_OFFS=$(cat "$CSR"  | openssl asn1parse -i | sed -n -E -e 's/^[[:blank:]]*([^:]*):.*BIT STRING.*/\1/p' | head -1)
PKID=$(cat "$CSR" | openssl asn1parse -i -strparse $SUBJECT_PUBLIC_KEY_OFFS -noout -out - | sha1sum -b | sed -n -E -e 's/^([[:alnum:]]+).*/\1/p')

# Login to vault
case $TARGET_VAULT in
    "prod")
        export VAULT_ADDR="https://vault.eove.fr:8200"
        ;;
    "preprod")
        export VAULT_ADDR="https://preprod.vault.eove.fr:8200"
        ;;
esac
vault login -method=github

# Check that this PKID comes from the vault
KEYS=$(vault list devices_pki/keys | sed -n -e '/^[[:xdigit:]][[:xdigit:]\-]*$/p')
for KEY in $KEYS
do
    if [[ $(vault read -format=json devices_pki/key/$KEY | jq -r '.data.subject_key_id' | sed -e 's/://g' ) == "$PKID" ]]
    then
        cat << EOF
🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉
CSR comes from vault
🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉
EOF
        exit
    fi
done

cat << EOF
💀💀💀💀💀💀💀💀💀💀💀💀💀
CSR was not found in vault
💀💀💀💀💀💀💀💀💀💀💀💀💀
EOF
exit -42
```

> [!Important]  
> The way vault computes the PKID is badly documented and may change in the future.
> This means that, if that script succeeds, you can trust that CSR.
> However, if it fails, then it can either be a wrong CSR **or** a change in vault. Please check accordingly.

Once the check have been performed, the IN65 is executed on the offline CA and the CSR is signed, we should get a certificate chain output PEM file, let's store it into `/tmp/online_cert.pem`.
We now enable the signed private/public key pair for online PKI by running the following script against the **online vault**:
```bash
./scripts/maintenance/import-signed-devices-certificate.sh /tmp/online_cert.pem
```

Once this has been done, in order to be sure that the imported certificate corresponds to a CSR generated by the online vault, please make sure that the current **default** issuer ID for the devices PKI has changed from the value you initially wrote down above:
```bash
vault read -format=json devices_pki/issuer/default/json | jq -r '.data.issuer_id'
```

> [!Important]  
> If the default issuer ID has not been updated, there may be a security risk. The imported PEM certificate could have been generated by another machine than the online vault and trust chain may be at risk.  
> This is even more important if the CSR PKID property (aka `subject_key_id`) could not be properly verified in the previous steps above.  
> In such situations, you should immediately [revoke (and publish) the PEM that the offline vault just signed](../../revocation.md). This requires running a new IN65.

### The offline CA can sign new online intermediate CAs for their whole lifetime during the next 12 months to come

The offline root CA signs offline intermediate CAs that last for many years.
Before the offline root CA becomes obsolete and thus cannot sign new intermediate CAs, a new offline root CA should be generated.

In such a case (a new offline root CA is generated), then all trust chain down to the device certification authority should be re-generated as well. This is tedious and should be done in advance (not in emergency), and well before the new offline root CA is actually used, so that all third parties (telemonitoring services) have time to update their list of trusted Eove CAs (add the new one alongside the existing one).

Only when all third parties are trusting the new chain, Eove will be free to swap the existing offline tree with a new one.

#### Test

Take the current date now, it will be called *D<sub>now</sub>*. Add 18 months to *D<sub>now</sub>*, then add again the duration of the online CA. The result is a date *D<sub>min</sub>*.

In order to get the expiry date of the offline CA in the trust chain used to sign devices, run the following command against the **online vault** after being logged in to the vault using an account with **admin** privileges:
```bash
vault read -format=json devices_pki/issuer/default/json | jq -r '.data.certificate' | openssl x509 -noout -text
```

Check the section called `Authority Information Access`, specifically the `CA Issuers` URI, that can be used to fetch the certificate of the offline CA that certifies the currently used online CA. We will store this URI in an environment variable `ONLINE_CA_ISSUER_URI` that we will use in the command below.

Download and dump the offline CA's certificate:
```
curl "$ONLINE_CA_ISSUER_URI" | openssl x509 -inform DER -noout -text
```

The expiry date will be called *D<sub>expiry</sub>*, it can be found in the `Validity`'s `Not After` attribute displayed by the command above.

The following statement should be true:
*D<sub>expiry</sub>*>*D<sub>min</sub>*.
If this is the case, this test is a PASS.  
If not, a new offline CA should be generated, the fix below should be applied.

#### Fix

We will need a new device signing offline CA, which also probably implies that a new root offline CA should be created as well.
You have two options there:
- Either the current offline root CA can be rotated (see https://developer.hashicorp.com/vault/tutorials/pki/pki-engine#step-7-rotate-root-ca)\
  This requires writing scripts for this to be run on the offline *ephemeral vault*.
- Or you can create a brand new offline root CA.\
  This would be a brand new start of the root CA and this means re-initializing a new vault, start from an empty backup etc.\
  Please read [the documentation on how to setup a new PKI](../offline/PKI_INit/md).

> [!Note]  
> In any case, the old PKI (and offline CAs) are still valid for at least 6 months, so you can use them up to the end of their validity. After their validity has elapsed, the expired CAs should be kept as read-only and won't be used anymore (except if revokation is required).

> [!Tip]  
> When renewing the root CA, you may as well evaluate the following aspects:
> - is the crypto used for the trust chain still up-to-date or should it be updated?
> - is hashicorp vault still up-to-date and maintained, in general and in NixOS or should the PKI be setup using new tools?
> - the environment in which the vault has been created initially has been progressively migrated from an initialization state 30 years before, it's maybe time to clean-up and start from scratch.
> - restarting from scratch allows to detach the currently active PKI from all history of previously chained reports and audit trails.
> - we may want to start with better security ecosystem (better crypto, improved initialisation in both hashicorp vault and our own scripts) after 30 years, it's probably time to review the whole setup in depth, including scripts, hashicorp vault, tools (hardware tokens, NixOS bootable media), and where and how backups+reports are saved.
> - after 30 years, it would be good to get new people own the whole system, including setup from scratch
> - you have 6 months...

### The next IN69 iteration is planned

A new IN69 should be planned within 1 year.
