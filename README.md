# openssl-cheatsheet

OpenSSL Commands Cheat Sheet

[![CC BY-NC-SA 4.0][cc-by-nc-sa-shield]][cc-by-nc-sa]

## Generate RSA Private Key

Create a private key with 2048 bits and save as `key.pem`.

```
openssl genrsa \
    -out key.pem \
    2048
```

Create a private key with 4096 bits encrypted with AES 256, retrieving password
from `password.txt` file and saving it in `key.pem`.

```
openssl genrsa \
    -aes256 \
    -passout file:password.txt \
    -out key.pem \
    4096
```

## RSA

Read a private key from `key.pem` and output it to _stdout_. This command will
validate if private key is valid.

```
openssl rsa \
    -in key.pem
```

Read a private key from _stdin_, don't output it but print its components.

```
cat key.pem \
    | openssl rsa \
        -noout -text
```

Convert a private key from PEM format to DER format. To convert from DER format
to PEM format, just swap parameters.

```
openssl rsa \
    -in key.pem \
    -inform PEM \
    -out key.der \
    -outform DER
```

Read a encrypted private key from `key.pem` using password from `password.txt`
file and output it to _stdout_.

```
openssl rsa \
    -in key.pem \
    -passin file:password.txt
```

Change a password from encrypted private key in `key-current.pem`, where current
password is in `password-current.txt` file and new password in
`password-new.txt` file, saving the new encrypted private key in `key-new.pem`
file.

```
openssl rsa \
    -in key-current.pem \
    -passin file:password-current.txt \
    -passout file:password-new.txt \
    -out key-new.pem
```

Extract public key from private key `key.pem` saving it in `pubkey.pem`.

```
openssl rsa \
    -in key.pem \
    -pubout \
    -out pubkey.pem
```

Read a public key from `pubkey.pem`, don't output it but print its components.

```
openssl rsa \
    -in pubkey.pem \
    -pubin \
    -noout -text
```

## Certificate Request

Configure OpenSSL certificate request to disable prompts, distinguished name
with predefined values and extended with subject alternative names (SANs).

```
cat > openssl.cnf <<EOS
[req]
prompt             = no
distinguished_name = req_distinguished_name
x509_extensions    = req_x509_extensions

[req_distinguished_name]
countryName            = BR
stateOrProvinceName    = SP
localityName           = Sao Paulo
organizationName       = John Doe LTDA
organizationalUnitName = Tecnologia da Informacao
commonName             = domain.tld
emailAddress           = john.doe@domain.tld

[req_x509_extensions]
subjectAltName = @req_x509_subjectAltName

[req_x509_subjectAltName]
DNS.1 = domain.tld
DNS.2 = *.domain.tld
EOS
```

Generate a new self-signed certificate configured by `openssl.cnf` valid by 365
from now, using private key `key.pem` file and outputing certificate to
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

Display subject alternative name extension from certificate in `cert.pem` file.

```
openssl x509 \
    -in cert.pem \
    -noout \
    -ext subjectAltName
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

## License

This work is licensed under a
[Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License][cc-by-nc-sa].

[![CC BY-NC-SA 4.0][cc-by-nc-sa-image]][cc-by-nc-sa]

[cc-by-nc-sa]: http://creativecommons.org/licenses/by-nc-sa/4.0/
[cc-by-nc-sa-image]: https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png
[cc-by-nc-sa-shield]: https://img.shields.io/badge/License-CC%20BY--NC--SA%204.0-lightgrey.svg
