Details matter.

---

When working on the date formatting portion of a global web app recently, I
really wanted to get two details right.

1. Localize the dates – People in different parts of the world are used to
seeing dates displayed differently. Give the people what they expect.
2. Four-digit years – Two-digit years are confusing. It's difficult enough to
parse "10/12/2011" when you don't know whether the month or the day comes first.
Parsing "10/12/11" just adds to the cognitive load.

I know the user's locale. PHP has [`IntlDateFormatter`][intl] to handle
localizing the dates. This should be easy.

But for space and legacy reasons, I needed a format like "10/11/2012". Well,
[`IntlDateFormatter::SHORT`][short] gives "10/11/12". That's _almost_ perfect.
_Almost_. You know, except for those pesky, confusing two-digit years.

I ran [a test][code-original] for `IntlDateFormatter::SHORT` for a handful of
different locales anyway. Maybe I'd get lucky.

```
en_US: 10/11/12
en_GB: 11/10/2012
de_DE: 11.10.12
fr_FR: 11/10/2012
ru_RU: 11.10.12
en_ZW: 11/10/2012
ps_AF: م. ۲۰۱۲/۱۰/۱۱
```

Kinda lucky. I guess. I mean, some of those locales nail it. But I need _every_
locale to have a four-digit year. And I certainly can't define every localized
format myself: I'm completely unfamiliar with how things are done in
Afghanistan, Zimbabwe, or the [200 or so states in between][states].

---

I took a walk.

On the walk, I came up with a plan.

---

The plan: `IntlDateFormatter` can [get][get] and [set][set] the date formatting
pattern. I can get the formatting pattern for a specific locale and coax it into
using a four-digit year. Good. But I needed to tread lightly; I didn't want to
mess with the date formatting pattern too much because I didn't want to, for
example, turn a Russian date into something Russians don't understand.

Treading lightly meant changing any occurrence of `yy` to `yyyy` but being extra
careful to only match _exactly_ two 'y's, not four. [Everybody Stand Back. I
know regular expressions.][xkcd]

After much fiddling – because who among us can really write a regex correctly on
the first try? – I ended up with
`preg_replace("/(?<!y)yy(?!y)/", "yyyy", $pattern)`. It [seems to work][regex].

And I ran [the test][code-modified] again.

```
en_US: 10/11/2012
en_GB: 11/10/2012
de_DE: 11.10.2012
fr_FR: 11/10/2012
ru_RU: 11.10.2012
en_ZW: 11/10/2012
ps_AF: م. ۲۰۱۲/۱۰/۱۱
```

Hooray! Localized dates with four-digit years. Exactly the details I need.

[intl]: http://php.net/manual/en/class.intldateformatter.php
[short]: http://php.net/manual/en/class.intldateformatter.php#intldateformatter.constants.short
[states]: https://en.wikipedia.org/wiki/List_of_sovereign_states
[get]: http://php.net/manual/en/intldateformatter.getpattern.php
[set]: http://php.net/manual/en/intldateformatter.setpattern.php
[xkcd]: http://xkcd.com/208/
[regex]: https://regex101.com/r/wD9qA1/2
[code-original]: https://gist.github.com/bryanburgers/f375ea3086a0ed029636#file-formattingtest-php-L5
[code-modified]: https://gist.github.com/bryanburgers/f375ea3086a0ed029636#file-formattingtest-php-L14
