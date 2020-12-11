---
layout: post
title: "CentOS 8 + Nginx + Phussion Passenger: unknown directive 'passenger_root'"
subtitle: ""
date: 2020-12-11
categories: [linux]
---

After following the passenger installation instruction from [https://www.phusionpassenger.com/library/install/nginx/install/oss/el7/](https://www.phusionpassenger.com/library/install/nginx/install/oss/el7/) I stubled upon an error:

```
nginx: [emerg] unknown directive "passenger_root" in /etc/nginx/conf.d/passenger.conf:1
```

It means that passenger module was probably not loaded correctly. First check where your nginx looks for module configs. It's usually written in `nginx.conf` and it may load all conf files from `module-enabled` or `modules.conf.d` directory, you can just grep your config to find out:

```
cd /etc/nginx
grep module *
```

Example output: `nginx.conf:include /etc/nginx/modules-enabled/*.conf;`

You have to create a new conf file there for loading the passenger module, but first find the module location:

```
find / -name ngx_http_passenger_module.so
```

Example output: `/usr/lib64/nginx/modules/ngx_http_passenger_module.so`

Now create new module configuration for phusion passenger and add above module there:

```
# cat /etc/nginx/modules-enabled/phusion-passenger.conf
load_module /usr/lib64/nginx/modules/ngx_http_passenger_module.so;
```

Restart Nginx 
```
sudo systemctl restart nginx
```
