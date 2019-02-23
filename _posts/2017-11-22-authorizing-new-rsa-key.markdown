---
layout: post
title: Authorizing new RSA key
subtitle: The less usual case
date: 2017-11-22
categories: [linux]
---
# The usual case

Most common case is that you have access to the linux host using username and password authentication. You have to generate new RSA key:

```bash
ssh-keygen -t rsa
```

Then the generated key has to be uploaded to the linux host, for example by:

```bash
ssh-copy-id -i <name_of_the_key.pub> <user@host>
```

And that's it. You can now establish a ssh connection using the RSA key, without being prompted for password:

```bash
ssh -i <path_to_private_key> <user>@<host>
```

# What if access to the host is only via another RSA key?

Recently I has another case: I had access to a list of hosts, but only using another RSA key. I wanted to add a new key to authorized keys list. Additionaly I wanted to do this operation for each of the host from the list. It turned out that `ssh-copy-id` can't handle it in such easy way as above. So I wrote a shell script, that adds given public key to the authorized_keys manually, for each host from the list:

```bash
#!/bin/sh

if [ "$#" -ne 2 ]; then
  echo 'Usage: add-key.sh <pub-key-file> <host1,host2,..,hostN>'
  exit -1
fi

PUB_KEY_FILE=$1
HOSTS=$2

IFS=',' read -ra ADDR <<< "$HOSTS"
for host in "${ADDR[@]}"; do
  echo "Adding $PUB_KEY_FILE to host $host"
  cat $PUB_KEY_FILE | ssh $host "cat >> ~/.ssh/authorized_keys"
done
```

Note: I have configured what key should be used in order to connect with given host in `~/.ssh/config`:

```
Host <host-addres>
  User root
  IdentityFile <path-to-private-key>
```

This makes `ssh <host>` works, without the need of manually specyfing the private key (using the `-i` switch). It is required for the script to work. 
