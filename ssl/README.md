# Generating Self-Signed Certificates

This is a long, involved process, but here's the high-level overview:

- generate a root key for the root certificate;
- use the root key to generate the root certificate;
- generate a server key for the server certificate;
- use the server key to configure a Certificate Signing Request (CSR) for the server certificate;
- use the CSR, the root key, and the root certificate to generate the server certificate;
- `cat` the server certificate and the root certificate to form a server certificate chain;
- use the server key and the server certificate chain in your `nginx` SSL server configuration;
- trust the root certificate on your local computer.

Whew!  Let's go through that step by step.  Fortunately, you'll only have to do it once.

## Generate the Root Key

```
cd ssl
openssl genrsa -out root.key 4096
```

## Generate the Root Certificate

```
openssl req \
  -x509 \
  -new \
  -nodes \
  -key root.key \
  -subj "/C=CA/ST=Ontario/L=Toronto/O=Localhost/CN=127.0.0.1" \
  -sha256 \
  -days 36500 \
  -out root.crt
```

## Generate the Server Key

```
openssl genrsa -out localhost.key 4096
```

## Generate the CSR

```
openssl req -new -sha256 -batch -key localhost.key -config localhost.cnf -out localhost.csr
```

## Generate the server certificate

```
openssl x509 \
  -req \
  -in localhost.csr \
  -extfile localhost.cnf -extensions req_ext \
  -CA root.crt -CAkey root.key -CAcreateserial \
  -out localhost.orig.crt \
  -days 36500 \
  -sha256
```

Note that we're creating this in `localhost.orig.crt`, because...

## Generate the server certificate chain

```
cat localhost.orig.crt root.crt > localhost.crt
```
