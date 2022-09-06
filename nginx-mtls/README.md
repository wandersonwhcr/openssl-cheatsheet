# nginx-mtls

## Certificate Authority

```
openssl genpkey \
    -algorithm RSA \
    -pkeyopt rsa_keygen_bits:2048 \
    -out cakey.pem
```

```
cat > openssl-ca.cnf <<EOS
[req]
prompt             = no
distinguished_name = req_distinguished_name

[req_distinguished_name]
organizationName = My Self-Signed Certificate Authority

[req_ext]
basicConstraints = @req_ext_basicConstraints

[req_ext_basicConstraints]
CA = TRUE
EOS
```

```
openssl req \
    -x509 \
    -config openssl-ca.cnf \
    -key cakey.pem \
    -days 365 \
    -out cacert.pem
```

## Server Certificate

```
openssl genpkey \
    -algorithm RSA \
    -pkeyopt rsa_keygen_bits:2048 \
    -out key.pem
```

```
cat > openssl.cnf <<EOS
[req]
prompt             = no
distinguished_name = req_distinguished_name
req_extensions     = req_ext # openssl req

[req_distinguished_name]
countryName         = BR
stateOrProvinceName = SP
localityName        = Sao Paulo
organizationName    = My Company
commonName          = mycompany.com

[req_ext]
subjectAltName = @req_ext_subjectAltName

[req_ext_subjectAltName]
DNS.1 = mycompany.com
DNS.2 = *.mycompany.com
EOS
```

```
openssl req \
    -new \
    -config openssl.cnf \
    -key key.pem \
    -out req.pem
```

```
openssl x509 \
    -req \
    -extfile openssl.cnf \
    -extensions req_ext \
    -in req.pem \
    -CA cacert.pem \
    -CAkey cakey.pem \
    -CAcreateserial \
    -days 365 \
    -out cert.pem
```
