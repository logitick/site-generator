---
title: "Add a TLS Certificate for Your Kubernetes Ingress"
date: 2019-04-25T21:42:23+08:00
tags: ["kubernetes", "k8s", "ssl", "tls", "tutorials"]
---

I keep forgetting how to do this. Now, I’m writing down because the internet
never forgets.

If you've been given a bunch of `crt` files here a the steps to add them to
kubernetes.

<!--more-->


## Bundle up the `.crt` files

If we're going to add the certificate to kubernetes we need to keep them
bundled up in one file. If you've got the already bundled up Certificate
Authority (CA) certificates, It's your lucky day. You'll only need to
concatenate you're domain's certificate with your CA certificate bundle.

```
$ cat example.com.crt >> fullchain.pem
$ cat My_CA_Bundle.ca-bundle >> fullchain.pem
```
Lucky bastard. You should [skip to adding the certificate to kubernetes]({{< relref "#create-a-kubernetes-tls-secret" >}})


### No CA Bundle
If you don't have the bundled CA certificates, you're going to have to
concatenate them yourself but you'll have to follow a specific order in order
to not break the chain.



Let's try and inspect the your domain's certificate.
```
$ openssl x509 -text -noout -in example.com.crt
```

It's going to spit out a lot of stuff but the important stuff in the first few lines

```
Certificate:
    Data:
    	...
        Issuer: C = US, O = Let's Encrypt, CN = Let's Encrypt Authority
    	...
        Subject: CN = example.com
    	...
```


The certificate issued for your domain always goes first. The next certificate
in the chain should have the `issuer` in the previous certificate as the
`subject`.

Basically, a valid full chain certificate should follow this pattern
```
Certificate:
    Data:
        Issuer: Issuer A
        Subject: CN = example.com
Certificate:
    Data:
        Issuer: Issuer B
        Subject: Issuer A
Certificate:
    Data:
        Issuer: ROOT Issuer
        Subject: Issuer B
Certificate:
    Data:
        Issuer: ROOT Issuer
        Subject: ROOT Issuer
```

After concatenating your certificates into a single file it's now ready to be
used by your kubernetes ingress.



## Create a Kubernetes TLS secret
We'll need to create a specific type of secret called `tls`. 
```bash
kubectl create secret tls <secret-name> --key <path-to-key> --cert <path-to-contatenated-certificates>

```

The `<path-to-key>` is the private key file which should have came with your
certificates.

In this case, I would run:
```bash
kubectl create secret tls www.example.com --key ./www_example_com.key --cert ./fullchain.pem
```


## Use the TLS secret in the ingress
Now all that's left to do is to call it from the ingress by its name.
```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: example-ingress
spec:
  rules:
  - host: www.example.com
    http:
      paths:
      - backend:
          serviceName: my-service
          servicePort: 80
  - host: example.com
    http:
      paths:
      - backend:
          serviceName: my-service
          servicePort: 80
  tls:
  - hosts:
    - www.example.com
    - example.com
    secretName: www.example.com
```
Depending on what you use for your ingress, It could take a bit of time before
it takes effect.


1. https://stackoverflow.com/questions/24153344/peformance-does-ssl-trust-chain-order-matter
2. http://tools.ietf.org/html/rfc5246#section-7.4.2
