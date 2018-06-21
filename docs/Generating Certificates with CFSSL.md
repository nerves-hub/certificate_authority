## Install dependencies

On Mac OS:

```sh
brew install cfssl
```

On other platforms `cfssl` can be installed from their github repo
https://github.com/cloudflare/cfssl

## Save config files

ca-config.json

```
{
  "signing": {
    "default": {
        "expiry": "730h",
        "usages": [
          "signing",
          "key encipherment"
      ]
    },
    "profiles": {
        "server": {
            "expiry": "730h",
            "usages": [
                "signing",
                "key encipherment",
                "server auth"
            ]
        },
        "client": {
            "expiry": "730h",
            "usages": [
                "signing",
                "key encipherment",
                "client auth"
            ]
        }
    }
  }
}
```

ca-csr.json

```
{
  "CN": "NervesHub Root CA",
  "key": {
    "algo": "ecdsa",
    "size": 256
  },
  "names": [
    {
      "O": "NervesHub",
      "OU": "NervesHub Certificate Authority"
    }
  ]
}
```

server.json

```
{
  "CN": "nerves-hub",
  "hosts": [
    "www.nerves-hub.org",
    "nerves-hub.local",
    "nerves-hub"
  ],
  "key": {
    "algo": "ecdsa",
    "size": 256
  },
  "names": [
    {
      "C": "US"
    }
  ]
}

```

## Generate SSL certificates

### Create certificate authority

```sh
cd certs
cfssl gencert -initca ca-csr.json | cfssljson -bare ca -
```

> If this were a real CA, `ca-key.pem` file should be kept in a safe place (i.e.
> not on a computer connected to the Internet)

### Server certificate

```sh
cd certs
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=www server.json | cfssljson -bare server
```

### Client certificate(s)

For as many devices as you want.  The `CN` will be passed to the server as the
device's serial number.

```sh
cd certs
echo '{"CN":"device-1234","hosts":[""],"key":{"algo":"ecdsa","size":256}}' | cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=client - | cfssljson -bare device-1234
```

## Generate SSL certificates for negative unit tests

These certificates are used to verify that bogus certificates are not allowed.

### Create fake certificate authority

```sh
cd certs
cfssl gencert -initca ca-csr.json | cfssljson -bare ca-fake -
```

### Create fake client certificate

```sh
cd certs
echo '{"CN":"device-fake","hosts":[""],"key":{"algo":"ecdsa","size":256}}' | cfssl gencert -ca=ca-fake.pem -ca-key=ca-fake-key.pem -config=ca-config.json -profile=client - | cfssljson -bare device-fake
```
