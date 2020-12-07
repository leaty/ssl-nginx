# ssl-nginx
Personal SSL guide for nginx

## Nginx conf
```nginx
listen 443 ssl;
ssl_certificate /etc/ssl/certs/example.com.crt;
ssl_certificate_key /etc/ssl/private/example.com.key;
```

### Security
It's best to generate a fitting one using: https://ssl-config.mozilla.org

## Digicert
### Generate new key (e.g. new server)
```console
openssl genrsa -out my.key 2048
```

### Generate csr using key
Use `-batch` to ignore csr options, since sites like digicert rewrite these anyway.
```console
openssl req -new -key my.key -out my.csr -batch
```

### Create duplicate on digicert
Simply put in the new or old csr and generate duplicate.

### Check cert info
```console
openssl x509 -in my.pem -text
```

## Self signed on fake domain (localhost, hosts file)
Example paths used below.
```console
CAkey=/etc/ssl/private/myCA.key
CApem=/etc/ssl/certs/myCA.pem
key=/etc/ssl/private/example.com.key
csr=/etc/ssl/certs/example.com.csr
crt=/etc/ssl/certs/example.com.crt
ext=/etc/ssl/certs/example.com.ext
```

### Become a CA
```console
openssl genrsa -out $CAkey 2048
openssl req -x509 -new -nodes -key $CAkey -sha256 -days 825 -out $CApem -subj "/CN=myCA" -batch
```

#### Trust that shit
```console
trust anchor $CApem
```

#### Remove trust (if you messed up)
```console
trust anchor --remove $CApem
```

### Generate certificate signing request

#### Generate key
```console
openssl genrsa -out $key 2048
```

#### Generate request
```console
openssl req -new -key $key -out $csr -subj "/CN=example.com" -batch
```

#### Extension file for SAN's ($ext)
```console
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
subjectAltName = @alt_names
[alt_names]
DNS.1 = example.com
DNS.2 = www.example.com
DNS.3 = test.example.com
DNS.4 = localhost
DNS.5 = localhost.localdomain
IP.1 = 127.0.0.1
IP.2 = ::1
```

### Generate signed certificate
```console
openssl x509 -req -in $csr -CA $CApem -CAkey $CAkey -CAcreateserial -out $crt -days 825 -sha256 -extfile $ext
```

### Verify names
```console
openssl verify -CAfile $CApem -verify_hostname example.com $crt
openssl verify -CAfile $CApem -verify_hostname www.example.com $crt
openssl verify -CAfile $CApem -verify_hostname test.example.com $crt
openssl verify -CAfile $CApem -verify_hostname localhost $crt
```

