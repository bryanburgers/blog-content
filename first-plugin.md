If you've never done it before, writing an ExpressionEngine add-on can be
daunting. First you have to figure out what type of add-on you need. Then you
need to make sure all the pieces are in just the right places.

![https://flic.kr/p/7DzC2S](/images/first-plugin/puzzle.jpg "Puzzle pieces")

A module needs control panel files, language files, updater files, all before
it even shows up in the list.  A fieldtype requires knowing exactly what
methods ExpressionEngine is going to call and how to get it all right. For an
extension, you're basically going to have to dig into ExpressionEngine core
code to have any chance of doing something worthwhile. And what even is an
Accessory? No, seriously. I don't know what an Accessory is.

And then there's the documentation. Kind of. All of these are half documented
and have a whole slew of unexpected sitautions that are not discussed in the
docs.

It's no wonder the first step into writing ExpressionEngine add-ons is scary.

## Enter: The Plugin

Enter: the Plugin. A breath of fresh air. If you're thinking about trying your
hand at add-on development, the Plugin is an easy, no-nonsense place to start.

First, it's easy to understand what it does. A plugin lets you put a custom
tag in your template, like `{exp:my_plugin:do_something}`.

Second, it only requires a single file. No updater file. No installation
process. No language file. Create `pi.my_plugin.php` and you're set.

Third, you probably won't even need to interact with the ExpressionEngine
monolith. No trying to remember how to load libraries or helpers or views.
Your plugin takes some input and gives some output. Easy.

## Let's do it

If it's so easy, let's do it.

First, open a template and type `{exp:my_plugin:hello}`.

Now, let's create the plugin. Go to `system/expressionengine/third_party` and
create a folder called `my_plugin`. Inside that folder, create
`pi.my_plugin.php`.

```php
<?php

$plugin_info = array(
  'pi_name'        => 'My Plugin',
  'pi_version'     => '1.0.0',
  'pi_author'      => 'ME!',
  'pi_author_url'  => 'http://burgers.io/',
  'pi_description' => 'My Plugin',
  'pi_usage'       => 'My first plugin! Use it to do stuff.'
);

class My_plugin {
  function hello() {
    return 'Hello, World';
  }
}
```

Boom. You created your first plugin. Go ahead, try it out.

## Input

Let's add a little spice in the form of a parameter. `{exp:my_plugin:hello
name="Bryan"}`.

```php
function hello() {
  $name = ee()->TMPL->fetch_param('name', 'World');
  return "Hello, {$name}";
}
```

Or, the alternative to a parameter is to use a tag pair.
`{exp:my_plugin:hello}Bryan{/exp:my_pluggin:hello}`.

```php
function hello() {
  $name = ee()->TMPL->tagdata;
  if ($name == '') {
    $name = 'World';
  }
  return "Hello, {$name}";
}
```

## That's it.

That's it. You've created a plugin. Easy to create. Easy to use. Powerful.

The best part? You finally took your first step into ExpressionEngine add-on
Development. It wasn't so scary, was it?
