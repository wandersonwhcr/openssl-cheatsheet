# mtls

This example shows how to configure nginx with mTLS to force TLS client
authentication using client certificate from same certificate authority from
server certificate.

## Certificate Authority

Generate a private key for certificate authority using RSA with 2048 bits and
output to `cakey.pem` file.

```
openssl genpkey \
    -algorithm RSA \
    -pkeyopt rsa_keygen_bits:2048 \
    -out cakey.pem
```

Create an OpenSSL x509 Config file `openssl-ca.cnf`.

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

Generate a self-signed certificate for certificate authority with
`openssl-ca.cnf` config file, using private key from `cakey.pem`, valid for 1
year and output its contents to `cacert.pem`.

```
openssl req \
    -x509 \
    -config openssl-ca.cnf \
    -key cakey.pem \
    -days 365 \
    -out cacert.pem
```

## Server Certificate

Generate a new private key for server, using RSA with 2048 bits. Output its
contents to `key.pem` file.

```
openssl genpkey \
    -algorithm RSA \
    -pkeyopt rsa_keygen_bits:2048 \
    -out key.pem
```

Create other OpenSSL x509 Config `openssl.cnf` file with information from server
certificate. Add subject alternative names for all required domains.

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

Create a certificate request using `openssl.cnf` config file with private key
`key.pem` and output its contents to `req.pem`.

```
openssl req \
    -new \
    -config openssl.cnf \
    -key key.pem \
    -out req.pem
```

Sign the certificate request `req.pem` using `openssl.cnf` config file adding
extensions from `req_ext` section, using certificate authority `cacert.pem`
certificate and `cakey.pem` private key, generating `cacert.srl` serial file if
it doesn't exists. Output certificate contents to `cert.pem`.

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

Configure nginx server to listen on port 443 with SSL with private key `key.pem`
and certificate `cert.pem`. Verify client certificate using certificate
authority `cacert.pem`. Output a JSON with `$HOSTNAME`.

```
cat > default.conf.template <<EOS
server {
    listen 443 ssl;

    ssl_certificate_key    /etc/ssl/private/key.pem;
    ssl_certificate        /etc/ssl/certs/cert.pem;
    ssl_client_certificate /etc/ssl/certs/cacert.pem;
    ssl_verify_client      on;

    default_type application/json;

    return 200 '{"hostname": "${HOSTNAME}"}';
}
EOS
```

Start server using Docker Compose adding volumes to mount private key `key.pem`,
server certificate `cert.pem`, certificate authority `cacert.pem` and nginx
config template `default.conf.template`.

```
docker-compose up \
    --detach
```

## Client Certificate

Generate a new private key for client using RSA with 2048 bits and output its
contents to `clkey.pem`.

```
openssl genpkey \
    -algorithm RSA \
    -pkeyopt rsa_keygen_bits:2048 \
    -out clkey.pem
```

Create last OpenSSL x509 Config `openssl-cl.cnf` file with client information.

```
cat > openssl-cl.cnf <<EOS
[req]
prompt             = no
distinguished_name = req_distinguished_name

[req_distinguished_name]
countryName         = BR
stateOrProvinceName = SP
localityName        = Sao Paulo
organizationName    = My Client
commonName          = myclient.com

[req_ext]
subjectAltName = @req_ext_subjectAltName

[req_ext_subjectAltName]
DNS.1 = myclient.com
EOS
```

Create a new certificate request for client using `openssl-cl.cnf` config file
and `clkey.pem` private key and save to `req.pem`.

```
openssl req \
    -new \
    -config openssl-cl.cnf \
    -key clkey.pem \
    -out clreq.pem
```

Sign the client certificate request `clreq.pem` using `openssl-cl.cnf` config
file adding extensions defined in `req_ext` section. Use certificate authority
`cacert.pem` certificate with `cakey.pem` private key, incrementing `cacert.srl`
serial. This client certificate is valid for 1 year and save it to `clcert.pem`.

```
openssl x509 \
    -req \
    -extfile openssl-cl.cnf \
    -extensions req_ext \
    -in clreq.pem \
    -CA cacert.pem \
    -CAkey cakey.pem \
    -CAcreateserial \
    -days 365 \
    -out clcert.pem
```

Make a HTTP Request to `https://mycompany.com` using `clkey.pem` client private
key and `clcert.pem` client certificate, verifying the certificate authority on
client side using `cacert.pem`. Resolve `mycompany.com:443` as localhost.

```
curl https://mycompany.com \
    --include \
    --cacert ./cacert.pem \
    --key ./clkey.pem \
    --cert ./clcert.pem \
    --resolve mycompany.com:443:127.0.0.1
```
