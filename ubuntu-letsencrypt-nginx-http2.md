I've been interested in [HTTP/2][h2] for a while. But until recently, I hadn't
had a chance to really play with it.

Then, within a month, two events that I had been waiting for both happened.

1. [Let's Encrypt][le] entered a [private beta][lepb].
2. [Nginx][nginx] released [HTTP/2 support][nginxh2].

I took it as a sign for me to secure this site with a Let's Encrypt certificate
and update it to use HTTP/2. This is how I made the transition.


## Step 1: Update nginx to 1.9.5

The first step was to update nginx to 1.9.5. I was previously on the version
that comes with Ubuntu 14.04 LTS.

So I found this article on [how to install nginx 1.9.5 on Ubuntu 14.04
LTS][nginxubuntu], and followed the directions.

But I made a mistake. In order to get this to work, I had to remove the old
nginx. And in the process of doing that, I managed to delete all of my nginx
configuration files. Oops!

Once I had restored those, `nginx -s reload` and I'm back in business.


## Step 2: Set up nginx to serve static files

Let's Encrypt doesn't support automatically getting a certificate using nginx,
yet. They do, however, support a method of verifying domain ownership by putting
a file in the root directory of a site. They call it the "webroot" plugin.

My previous nginx configuration was set up to proxy all of the traffic to a
backend node app.

```
location / {
    proxy_pass http://nodeapp;
}
```

But to be able to point Let's Encrypt at the root directory, I needed to _have_
a root directory on the filesystem. So I updated my configuration to allow that.

```
location / {
    root /var/www/burgers.io/static;
    try_files $uri @proxy;
}
location @proxy {
    proxy_pass http://nodeapp;
}
```

What this does is that, when a request comes in, nginx first checks if the file
exists in my static directory, and if it does, it serves the file. If it
doesn't, it passes the request off to my node app.


## Step 3: Run Let's Encrypt

Now, I needed to actually get the certificates using Let's Encrypt. I took the
instructions that were sent to me in my limited beta invite and used the
"webroot" plugin.

```
git clone https://github.com/letsencrypt/letsencrypt
cd letsencrypt
./letsencrypt-auto certonly -a webroot \
    --webroot-path /var/www/burgers.io/static \
    -d burgers.io \
    -d www.burgers.io \
    --server https://acme-v01.api.letsencrypt.org/directory \
    --agree-dev-preview \
    --rsa-key-size 4096
```

I was surprised by two things.

1. That it worked so well for limited beta software.
2. That it installed all of the packages I needed to complete the task without
   asking me. This was a little disconcerting.

But it did work, and I had my certs.


## Step 4: Set up securely and enable HTTP/2

I then followed [this article on securing nginx][leb] to set up my TLS securely
using the keys that were generated.

And with that done, enabling HTTP/2 was as simple as `listen 443 ssl http2`.

Run an `nginx -s reload` and I now have a blog secured with a Let's Encrypt certificate
serving HTTP/2 to browsers that support it. Hooray!


## Step 5: Google Calendar alert

The Let's Encrypt limited beta doesn't look like it will continuously update my
TLS Certificate. And because of their [90-day certificate lifetime][90day], I
need to remember to refresh this certificate in about 60 days.

So the last step was to add a calendar alert to my calendar, and hope that by
then Let's Encrypt is out of beta and can continuously keep my certificate
up-to-date.

---

I was expecting a lot more work, but, with the help of a bunch of existing
tutorials, I was able to get nginx serving HTTP/2 and a Let's Encrypt
certificate on my Ubuntu 14.04 server in a couple of hours.


[h2]: https://http2.github.io/
[nginx]: https://www.nginx.com/
[nginxh2]: https://www.nginx.com/blog/nginx-1-9-5/
[le]: https://letsencrypt.org/
[lepb]: https://www.eff.org/deeplinks/2015/10/lets-encrypt-enters-private-beta
[nginxubuntu]: https://by-example.org/install-nginx-with-http2-support-on-ubuntu-14-04-lts/
[leb]: https://www.wjd.io/lets-encrypt-beta
[90day]: https://letsencrypt.org/2015/11/09/why-90-days.html
