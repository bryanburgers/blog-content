[Stash][stash], created by Mark Croxton, is a powerful tool for templating in
ExpressionEngine. Used the right way, it can improve the readability of
templates.

> Any fool can write code that a computer can understand. Good programmers
> write code that humans can understand.
> – Martin Fowler

Used the wrong way, Stash could choke a horse.

<blockquote class="twitter-tweet" lang="en"><p>I’ve never seen an add-on as abused and overused as Stash. These templates could choke a horse. <a href="https://twitter.com/hashtag/eecms?src=hash">#eecms</a></p>&mdash; Ryan Masuga (@masuga) <a href="https://twitter.com/masuga/status/563168454929575936">February 5, 2015</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

I'll admit I don't actually know what that expression means, but I understand
the sentiment. I, too, have seen Stash used in a way that makes templates
completely unintelligible. And for what?

I present here the way I use Stash to create what I hope are readable
templates.


## Why Stash?

So why am I using Stash in the first place?

* To organize my templates and reduce repeated code.
* To preload data, for example to prevent nesting `{exp:channel:entries}`
  tags.
* To cache pages.


## Holes

The sites I work on have up to a dozen different channels, all with unique
templates. But at the core, they all share a consistent HTML structure.

My layout files – I say "files" but I frequently only use a single file –
define this consistent structure, and leave holes for each template to express
its individuality.

Holes. Yep. That's the technical term I use.

The layout defines the common site structure, and then leaves holes for each
template to fill.

Enough talk. Example time.

```html
<!doctype html>
<html>
<head>
	{snippet:html-head}
</head>
<body>
	<div class="body-container">
		{snippet:header}

		<div>
			<main role="main" class="page-main">
				<header class="page-title">
					<h1>{layout:title}</h1>
				</header>

				<div class="page-main-container">
					{if layout:breadcrumbs}
					<nav class="nav-breadcrumb">
						<ul>
							{layout:breadcrumbs}
						</ul>
					</nav>
					{/if}

					<section class="page-content">
						{layout:content}
					</section>
				</div>
			</main>

			{if layout:sidebar}
			<aside class="page-sidebar">
				{layout:sidebar}
			</aside>
			{/if}
		</div>

		{snippet:footer}
		{snippet:scripts}

	</div>
</body>
</html>
```

All of my holes are named `{layout:something}`. I often have enough things
called "content" or "title" that I need to have a prefix to keep everything
straight.


## Filling Holes

With the layout (or layouts) defined, the templates are responsible for
choosing which layout to use, and to fill the holes in that template.

```html
{exp:stash:cache replace='{global:stash_replace}'}
{stash:embed:global process='inline'}
{stash:embed:layouts:page}

{exp:channel:entries limit="1" disable='member_data|pagination|categories'}

{!-- HTML Header items --}
{exp:stash:set scope='local' type='snippet'}
	{stash:page:meta:title}{title} | {global_site_name}{/stash:page:meta:title}
	{stash:page:meta:description}{page_seo_description}{/stash:page:meta:description}

	{stash:page:meta:url}{page_url}{/stash:page:meta:url}
	{stash:page:meta:social:title}{title}{/stash:page:meta:social:title}
	{stash:page:meta:social:description}{page_excerpt}{/stash:page:meta:social:description}
{/exp:stash:set}

{!-- Layout holes --}
{exp:stash:set scope='local' type='snippet'}

	{stash:layout:title}{title}{/stash:layout:title}

	{stash:layout:content}
		{if page_header_image}
		<img class="featured" src="{page_header_image:header}" alt="{page_header_image_alt}">
		{/if}

		{page_content}
	{/stash:layout:content}

	{stash:layout:sidebar}
		{stash:embed:shared:standard-sidebar-navigation}
		{snippet:email-signup}
	{/stash:layout:sidebar}

	{stash:layout:breadcrumbs}
		<li><a href="/"><i class="fa fa-home"></i></a></li>
		{exp:structure:breadcrumb inc_home="no" include_ul="no" here_as_title="yes" wrap_each="li" separator=""}
	{/stash:layout:breadcrumbs}

{/exp:stash:set}

{/exp:channel:entries}
{/exp:stash:cache}
```

This method leaves each template with the flexibility to do what it needs to
do, all the while staying within the structure of the site.


## The homepage

One last consideration is the homepage. The homepage of a site is almost
always unique. While it shares a visual theme with the rest of the site, the
structure is often very different.

So for the homepage, I don't use a layout. Splitting it out into a layout and
a template doesn't have benefits. Because the homepage is unique, the layout
will never be reused.

I keep all of the homepage in a single file. That way I only have one place to
look when I need to change it.


## Why not viewmodels?

I tried [viewmodels][vm] for a while. I gave it my best shot. But every time I
did, I felt it was adding a layer of indirection that wasn't giving me
anything in return.

Any time I needed to make a change, I needed to change both the view and the
model. I'd end up touching both files anyway, and instead of looking in one
place I'd have to look in two places.


## Why not native layouts?

Everything I've done in this article can probably be done using
ExpressionEngine's [native layouts][native].

I haven't used native layouts, but I'm sure they're great. Stash just has
enough other small, useful features that I continue to use it.


---


The way I use [Stash][stash] certainly isn't the one true way. But it's worked
very well for me and for my coworkers. Let me know what you think; I'd be
curious to hear any feedback on this approach.


[stash]: https://github.com/croxton/stash
[vm]: https://github.com/croxton/Stash/wiki/Template-inheritance
[native]: https://ellislab.com/expressionengine/user-guide/templates/layouts.html
