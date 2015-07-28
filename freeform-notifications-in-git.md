Where I work, our ExpressionEngine deployment process is usually one step: `git
push origin master`. I love that it's so easy.

When it isn't that easy, when there are extra steps, it's too easy to forget the
extra steps (and I get annoyed). Sometimes these extra steps are unavoidable: a
channel field needs to be added, a module needs to be installed, etc.

Recently I ran across an annoying extra step that could be automated; I just had
to think through it: updating Freeform notification templates.

Freeform does not allow saving its notification templates to the disk. That
means that they can't be tracked in git. There doesn't seem to be any way around
this. Which, for us, previously meant copy/pasting an updated notification
template from a dev or staging environment to the production environment.
Annoying. Easy to forget.

However, we already track normal ExpressionEngine templates in git. If we could
store notification templates as ExpressionEngine templates, we could track them
in git and avoid an unnecessary deployment step.

That line of thinking led to this plan:

1. Create an ExpressionEngine template using `{exp:freeform:entries}`.
2. Call that ExpressionEngine template from the Freeform notification template.


## Step 1: Create an ExpressionEngine template

In a Freeform notification template, all of the variables are in the easy-to-use
`{first_name}` format. In an ExpressionEngine template, they are not.

To get that data, we need to use the `{exp:freeform:entries}` tag with the right
parameters.

```
{exp:freeform:entries
    dynamic="no"
    form_name="some-form"
    status="open|closed|pending"
    entry_id="{embed:entry_id}"}

{/exp:freeform:entries}
```

(Note that, in my testing, the `form_name` parameter was **required** for this
to work.)

To get the data in this loop, we can use `{freeform:field:first_name}` instead
of `{first_name}`. (A list of all of the variables and tag pairs is available in
the [Freeform documentation][ffentries].)

With this structure in place, we can create a (very crude) notification template
like this:

```html
{exp:freeform:entries
    dynamic="no"
    form_name="some-form"
    status="open|closed|pending"
    entry_id="{embed:entry_id}"}
  <h1>Somebody submitted a form...</h1>
  <dl>
    <dt>Name</dt>
    <dd>{freeform:field:name}</dd>
    <dt>Email</dt>
    <dd>{freeform:field:email}</dd>

    {freeform:all_form_fields}
      <dt>{freeform:field_label}</dt>
      <dd>{freeform:field_output}</dd>
    {/freeform:all_form_fields}
  </dl>
{/exp:freeform:entries}
```


## Step 2: Put an embed in the notification template

Because Freeform notifications can't be tracked, we want to put as little in the
Freeform notification template as possible. And never have to change it again.

The minimal stub that worked best for us is just an embed with an entry_id
parameter.

```
{embed="freeform-notifications/.some-notification"
    entry_id="{freeform_entry_id}"}
```

---

That's it. Changes to notification templates happen in the ExpressionEngine
template, are tracked in git, and get deployed.

That's one less thing for me to forget during deployment.




[ffentries]: https://www.solspace.com/docs/freeform/entries/
