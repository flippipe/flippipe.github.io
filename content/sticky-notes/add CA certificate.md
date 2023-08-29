---
title: "Add CA Certificate"
date: 2023-08-29T17:11:40+01:00
summary: "Add certificate to the linux trust store"
description: "Add certificate to the linux trust store"
tags: ["CA", "certificates"]
draft: false
---

Sometimes, we need to add self-signed certificated to the CA trust list of our server for reasons. And I never remember which path or command in each linux family.

``` sh
#!/bin/bash
export fqdn="example.com"

openssl s_client -showcerts -servername "${fqdn}" -connect "${fqdn}":443 </dev/null | sed -n -e '/-.BEGIN/,/-.END/ p' > "${fqdn}.crt"

os_like=$(grep ID_LIKE /etc/os-release | cut -d= -f 2 | tr -d '"')

if [ "${os_like}" = "debian" ]; then
    # debian is picky in the extension used
    cp "${fqdn}.crt" /usr/local/share/ca-certificates/
    update-ca-certificates
elif [ "${os_like}" = "fedora" ]; then
    cp "${fqdn}.crt" /etc/pki/ca-trust/source/anchors/
    update-ca-trust
fi

```

Thanks for [Sorin Zamfir](https://www.baeldung.com/linux/ssl-certificates#bd-using-openssl) where i stole sed command to strip non useful stuff.