# openssl-cheatsheet

OpenSSL Commands Cheat Sheet

[![CC BY-NC-SA 4.0][cc-by-nc-sa-shield]][cc-by-nc-sa]

## Generate Private Key

Create a private key using RSA with 2048 bits and save as `key.pem`.

```
openssl genpkey \
    -algorithm RSA \
    -pkeyopt rsa_keygen_bits:2048 \
    -out key.pem
```

```
openssl genrsa \
    -out key.pem \
    2048
```

Create a private key using RSA with 4096 bits encrypted with AES 256, retrieving
password from `password.txt` file and saving it in `key.pem`.

```
openssl genpkey \
    -algorithm RSA \
    -pkeyopt rsa_keygen_bits:4096 \
    -aes256 \
    -pass file:password.txt \
    -out key.pem
```

```
openssl genrsa \
    -aes256 \
    -passout file:password.txt \
    -out key.pem \
    4096
```

## Private Key

Read a private key from `key.pem` and output it to _stdout_.

```
openssl pkey \
    -in key.pem
```

```
openssl rsa \
    -in key.pem
```

Read a private key from _stdin_, don't output it but print its components.

```
cat key.pem \
    | openssl pkey \
        -noout -text
```

```
cat key.pem \
    | openssl rsa \
        -noout -text
```

Convert a private key from PEM format to DER format. To convert from DER format
to PEM format, just swap parameters.

```
openssl pkey \
    -in key.pem \
    -inform PEM \
    -out key.der \
    -outform DER
```

```
openssl rsa \
    -in key.pem \
    -inform PEM \
    -out key.der \
    -outform DER
```

Create an encrypted private key with AES256 from `key.pem` using password from
`password.txt` and output it to _stdout_.

```
openssl pkey \
    -in key.pem \
    -aes256 \
    -passout file:password.txt
```

```
openssl rsa \
    -in key.pem \
    -aes256 \
    -passout file:password.txt
```

Read an encrypted private key from `key.pem` using password from `password.txt`
file and output it to _stdout_.

```
openssl pkey \
    -in key.pem \
    -passin file:password.txt
```

```
openssl rsa \
    -in key.pem \
    -passin file:password.txt
```

Change a password from encrypted private key in `key.pem`, where current
password is in `password.txt` file and new password in `passwordout.txt` file,
saving the new encrypted private key with AES256 in `keyout.pem` file.

```
openssl pkey \
    -in key.pem \
    -passin file:password.txt \
    -aes256 \
    -passout file:passwordout.txt \
    -out keyout.pem
```

```
openssl rsa \
    -in key.pem \
    -passin file:password.txt \
    -aes256 \
    -passout file:passwordout.txt \
    -out keyout.pem
```

Extract public key from private key `key.pem` saving it in `pubkey.pem`.

```
openssl pkey \
    -in key.pem \
    -pubout \
    -out pubkey.pem
```

```
openssl rsa \
    -in key.pem \
    -pubout \
    -out pubkey.pem
```

Read a public key from `pubkey.pem`, don't output it but print its components.

```
openssl pkey \
    -in pubkey.pem \
    -pubin \
    -noout -text
```

```
openssl rsa \
    -in pubkey.pem \
    -pubin \
    -noout -text
```

## Certificate Request

Configure OpenSSL certificate request to disable prompts, distinguished name
with predefined values and extended with subject alternative names (SANs). If CA
certificate is needed uncomment basic constraint.

```
cat > openssl.cnf <<EOS
[req]
prompt             = no
distinguished_name = req_distinguished_name
req_extensions     = req_ext # openssl req
x509_extensions    = req_ext # openssl req -x509

[req_distinguished_name]
countryName            = BR
stateOrProvinceName    = SP
localityName           = Sao Paulo
organizationName       = John Doe LTDA
organizationalUnitName = Tecnologia da Informacao
commonName             = domain.tld
emailAddress           = john.doe@domain.tld

[req_ext]
subjectAltName   = @req_ext_subjectAltName
#basicConstraints = @req_ext_basicConstraints

[req_ext_subjectAltName]
DNS.1   = domain.tld
DNS.2   = *.domain.tld
IP.1    = 127.0.0.1
IP.2    = 192.168.0.1
email.1 = john.doe@domain.tld

[req_ext_basicConstraints]
CA = TRUE
EOS
```

Create a certificate request configured by `openssl.cnf` using private key
`key.pem` file and outputing request to `req.pem`.

```
openssl req \
    -new \
    -config openssl.cnf \
    -key key.pem \
    -out req.pem
```

Read certificate request from `req.pem`, don't output it but print its
components.

```
openssl req \
    -in req.pem \
    -noout -text
```

Generate a new self-signed certificate configured by `openssl.cnf` valid by 365
days from now, using private key `key.pem` file and outputing certificate to
`cert.pem`.

```
openssl req \
    -x509 \
    -config openssl.cnf \
    -days 365 \
    -key key.pem \
    -out cert.pem
```

## Certificate

Read certificate from `cert.pem`, don't output it but print its components.

```
openssl x509 \
    -in cert.pem \
    -noout -text
```

Print certificate purposes or where it can be used from `cert.pem`.

```
openssl x509 \
    -in cert.pem \
    -noout -purpose
```

Show certificate validity only from `cert.pem`.

```
openssl x509 \
    -in cert.pem \
    -noout -dates
```

Read certificate from `cert.pem` and show only its subject and issuer.

```
openssl x509 \
    -in cert.pem \
    -noout \
    -subject -issuer
```

Read certificate from `cert.pem`, don't output it but print its components,
disabling public key and signature.

```
openssl x509 \
    -in cert.pem \
    -certopt no_pubkey \
    -certopt no_sigdump \
    -noout -text
```

Display subject alternative name and basic constraints extensions from
certificate in `cert.pem` file.

```
openssl x509 \
    -in cert.pem \
    -noout \
    -ext subjectAltName,basicConstraints
```

Show fingerprints from certificate in `cert.pem`. This can be used to compare
certificates.

```
openssl x509 \
    -in cert.pem \
    -noout \
    -fingerprint -md5

openssl x509 \
    -in cert.pem \
    -noout \
    -fingerprint -sha1

openssl x509 \
    -in cert.pem \
    -noout \
    -fingerprint -sha256
```

Extract from certificate `cert.pem` its public key and save to `pubkey.pem`.

```
openssl x509 \
    -in cert.pem \
    -noout \
    -pubkey \
    -out pubkey.pem
```

Sign a certificate request from `req.pem` file using certificate authority from
`cacert.pem` and private key `cakey.pem`, creating a serial `cacert.srl` file if
it doesn't exist and output results to `cert.pem` file. This subcommand acts
like a "mini CA".

```
openssl x509 \
    -req \
    -extfile openssl.cnf \
    -extensions req_ext \
    -in req.pem \
    -CA cacert.pem \
    -CAkey cakey.pem \
    -CAcreateserial \
    -out cert.pem
```

Check if a certificate `cert.pem` is issued by a certificate authority
`cacert.pem`.

```
openssl verify \
    -CAfile cacert.pem \
    cert.pem
```

## SSL Client

Retrieve certificate from `domain.tld` on port `443`. By default, this
subcommand expect user input and to disable it, use `/dev/null` as _stdin_. Some
debugging information is sent to _stderr_ and to avoid output redirect this file
descriptor to `/dev/null`.

```
openssl s_client \
    -connect domain.tld:443 \
    < /dev/null 2> /dev/null
```

Retrieve certificate from `domain.tld` as server name `subdomain.domain.tld`.

```
openssl s_client \
    -connect domain.tld:443 \
    -servername subdomain.domain.tld \
    < /dev/null 2> /dev/null
```

Retrieve certificate from `domain.tld` and output its components.

```
openssl s_client \
    -connect domain.tld:443 \
    < /dev/null 2> /dev/null \
    | openssl x509 \
        -noout -text
```

## PKCS#12

Create PKCS#12 `cert.p12` using private key from `key.pem` file, certificate
from `cert.pem` and additional certificate authority certificate from
`cacert.pem` file. Encrypt file using retrieved password from `password.txt`
file.

```
openssl pkcs12 \
    -export \
    -inkey key.pem \
    -in cert.pem \
    -certfile cacert.pem \
    -out cert.p12 \
    -passout file:password.txt
```

Read a PKCS#12 `cert.p12` file, decrypt using password from `password.txt` file
and output results to _stdout_ without encrypt private key.

```
openssl pkcs12 \
    -in cert.p12 \
    -passin file:password.txt \
    -nodes
```

Read a PKCS#12 `cert.p12` file, decrypt using password from `password.txt` file
and output to _stdout_ only the private key without encryption.

```
openssl pkcs12 \
    -in cert.p12 \
    -passin file:password.txt \
    -nodes \
    -nocerts
```

Read a PKCS#12 `cert.p12` file, decrypt using password from `password.txt` file
and output to _stdout_ only client certificates.

```
openssl pkcs12 \
    -in cert.p12 \
    -passin file:password.txt \
    -clcerts \
    -nokeys
```

Read a PKCS#12 `cert.p12` file, decrypt using password from `password.txt` file
and output to _stdout_ only certificate authority certificates.

```
openssl pkcs12 \
    -in cert.p12 \
    -passin file:password.txt \
    -cacerts \
    -nokeys
```

## Digest

Create a MD5 message digest for `file.txt` file.

```
openssl dgst \
    -md5 \
    file.txt
```

Create a SHA256 message digest for `file.txt` and output results in a format to
use with `sha256sum`.

```
openssl dgst \
    -sha256 -r \
    file.txt
```

## Encrypt Symmetric

Encode file `file.txt` to Base64 and save results to `file.txt.enc`.

```
openssl enc \
    -in file.txt \
    -base64 \
    -out file.txt.enc
```

Decode file `file.txt.enc` from Base64 and output results to _stdout_.

```
openssl enc \
    -in file.txt.enc \
    -base64 -d
```

## Encrypt Asymmetric

Using RSA, encrypt a `file.txt` file using the public key `pubkey.pem` and
output encrypted content to `file.txt.enc` file.

```
openssl rsautl \
    -encrypt \
    -in file.txt \
    -inkey pubkey.pem \
    -pubin \
    -out file.txt.enc
```

Using RSA, decrypt a `file.txt.enc` file using the private key `key.pem` and
output raw content to `file.txt` file.

```
openssl rsautl \
    -decrypt \
    -in file.txt.enc \
    -inkey key.pem \
    -out file.txt
```

Encrypt a `file.txt` file using certificate `cert.pem` that contains a RSA
public key and output encrypted content to `file.txt.enc`.

```
openssl rsautl \
    -encrypt \
    -in file.txt \
    -inkey cert.pem \
    -certin \
    -out file.txt.enc
```

## Random

Generate 10 pseudo random bytes and display it as base64.

```
openssl rand \
    -base64 \
    10
```

Generate random string with 40 hexadecimal chars. Each byte is displayed as two
hexadecimal character, so 20 bytes will display 40 chars.

```
openssl rand \
    -hex \
    20
```

## Tips

OpenSSL is built as command and subcommand.

```
openssl subcommand
```

Every subcommand has its own man page.

```
man openssl-subcommand
```

Every argument is defined using a single dash.

```
openssl subcommand -argument
```

Passwords can be read from many sources.

```
openssl subcommand -passin pass:password
openssl subcommand -passin file:password.txt
openssl subcommand -passin env:PASSWORD
openssl subcommand -passin fd:3
openssl subcommand -passin stdin
```

## References

* [OpenSSL x509v3_config](https://www.openssl.org/docs/man3.0/man5/x509v3_config.html)

## License

This work is licensed under a
[Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License][cc-by-nc-sa].

[![CC BY-NC-SA 4.0][cc-by-nc-sa-image]][cc-by-nc-sa]

[cc-by-nc-sa]: http://creativecommons.org/licenses/by-nc-sa/4.0/
[cc-by-nc-sa-image]: https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png
[cc-by-nc-sa-shield]: https://img.shields.io/badge/License-CC%20BY--NC--SA%204.0-lightgrey.svg
