---
layout: post
title: "Nginx proxy pass with https"
subtitle: ""
date: 2021-06-06
categories: [programming]
---

Recently I deployed a web app and configured nginx proxy pass for it. I served it over https, the configuration looked similar to this:
```
location / {                                                                                                                                                                                                                                                                                                                                                                            
  proxy_pass http://127.0.0.1:<port>;
  proxy_set_header Host $host;
  proxy_set_header X-Real-IP $remote_addr;
  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
}
```

And it worked ..almost. I could login on firefox, but coud not on chrome. What's wrong with you chrome?! I could see the certificate is valid, no errors in console or network traffic. Then I noticed an error in "Issues" tab in chrome developer tools:

![Migrate entirely to HTTPS to allow cookies to be set by same-site subresources](/images/https_cookie_issue.png){: .center-image}

It turned out that while accessing the page worked over https, all the requests proxied to the backed were non encrypted - fixed it by adding 

```
proxy_set_header X-Forwarded-Proto $scheme;
```

to the nginx proxy pass config.
