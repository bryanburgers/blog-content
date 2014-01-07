Far-future cache headers are important for the performance of a site. Plenty
of advice can be found on the internet about why you should set the cache
headers for your static assets to a year in the future. But with far-future
cache headers comes the need to be able to *bust the cache* when a file
changes.

My favorite method of cache busting is the [method espoused by Steve Sounders](http://www.stevesouders.com/blog/2008/08/23/revving-filenames-dont-use-querystring/)
and used in [HTML5 Boilerplate](https://github.com/h5bp/html5-boilerplate/blob/master/.htaccess#L598)
that rewrites a URL like `/js/script.20140107.js` to the file at
`/js/script.js`.

I actually use a slightly modified rewrite of the H5BP method that allows the
`a-f` characters. It's a small change, but that lets me use the SHA1 of the
file as cache busting. I use this so much at work, in fact, that I wrote an
ExpressionEngine [plugin called Buster](https://github.com/click-rain/buster)
to do exactly that: calculate the SHA1 of a static asset and stick it into
that asset's URL.

In Apache's configuration, it basically looks like this.

```
<IfModule mod_rewrite.c>
  RewriteEngine On

  RewriteCond %{REQUEST_FILENAME} !-f
  RewriteCond %{REQUEST_FILENAME} !-d
  RewriteRule ^(.+)\.([0-9a-f]+)\.(js|css|png|jpg|gif|svg)$ $1.$3 [L]
</IfModule>
```

But this post isn't about Apache configs or ExpressionEngine. It's about
Node.js.

When I started using Node.js, I missed my standard Apache rewrite that allowed
me to easily invalidate cached assets. Call me blunt, but I didn't see a good
module on NPM to do this, and it frustrated me.

It wasn't until much later that I realized there wasn't a good module on NPM
for this because there's no need for one. All you need for good URL rewriting
is – hold on to your hats, this may come as a surprise – to rewrite the URL.

The strategy, then, is to rewrite the URL before it gets to the code that
services the static assets with far-future caching. In a recent project, I
used the [node-static](https://github.com/cloudhead/node-static) module to
serve my static assets.

```js
var file = new(static.Server)('./public', { cache: 86400*365, gzip: true });

var server = http.createServer(function(req, res) {

  // ... other logic ...

  // Rewrite the URL before it gets to node-static.
  req.url = req.url.replace(/\/([^\/]+)\.[0-9a-f]+\.(css|js|jpg|png|gif|svg)$/, "/$1.$2");

  req.addListener('end', function () {
    //
    // Serve files!
    //
    file.serve(req, res);
  }).resume();
});
```

Here, node-static expects the actual name of the file to serve. But before the
URL even gets to node-static, we simply rewrite the URL from `asset.a35f69.js`
to `asset.js`, which is exactly what node-static wants. It never even sees the
cache-busting junk in the middle.

I've also used [Express](http://expressjs.com/) to build websites in Node.js,
and the concept is the same. Before the URL gets to Express' static
middleware, we need to rewrite the URL.

```js
var app = express();
// Rewrite the URL before it gets to Express' static middleware.
app.use('/public/', function(req, res, next) {
        req.url = req.url.replace(/\/([^\/]+)\.[0-9a-f]+\.(css|js|jpg|png|gif|svg)$/, "/$1.$2");
        next();
});
app.use('/public/', express['static'](__dirname + '/public', { maxAge: 30 }));
```

It just takes a line or two using either Node's raw HTTP server or Express to
rewrite URLs for static assets. No NPM module required.
