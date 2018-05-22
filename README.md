# plain
plain templates

A simple approach to making a template is this:

``` html
<div class="contact template">
  <div class="name">Jamie Douglas</div>
  <div class="phone">(321) 522-1937</div>
</div>
```

``` css
.template {
  display: none;
}
```

``` js
function addContact(name, phone) {
  const elem = $('.template.contact').clone().removeClass('template');
  $('.name', elem).text(name);
  $('.phone', elem).text(phone);
  elem.appendTo($('.contacts'));
}
```

This is kinda cool because it makes a template from a real example. However, this
can cause data leakage if using real data, or example leakage if using placeholder
examples, if a mistake is made when creating the template or if the HTML part of
the template is updated without updating the JavaScript part of it. For instance
if the HTML is changed to this:

``` html
<div class="contact template">
  <div class="name">Jamie Douglas</div>
  <div class="phone">(321) 522-1937</div>
  <div class="email">inside.joke.from.the.devs@example.com</div>
</div>
```

...but the JavaScript isn't updated, all contacts added to the list with `addContact`
will have the example email address.

This can easily happen, especially if the template HTML is stored in a separate file
from the template JavaScript.

plain aims to fix both of these problems by adding a SHA of the template text to the
template data, which may be stored in a different file. This way it will be unweildly
to create a template without using an authoring tool, and the authoring tool will make
it clear what text will be replaced and will never show up when rendering a template.
It will also make it unweildly to go in and edit the template text without using the
authoring tool because the template will refuse to render if the SHA in the json
doesn't match the SHA in the contact. Here's an example:

``` html
<div class="contact">
  <div class="name">Jamie Douglas</div>
  <div class="phone">(321) 522-1937</div>
</div>
```

``` json
{
  "data": [
    {
      "selector": ".name",
      "param": "name"
    },
    {
      "selector": ".phone",
      "param": "phone"
    }
  ],
  "checksum": "3382eb9862a74aa642f51514fb1fc2148a9fd693"
}
```

``` javascript
import Template from '@apicomponents/plain'
import contactTemplate from './templates/contact.html' // or fs.readFileSync()
import contactTemplateData from './templates/contact.json'

// this will throw an error if the checksum doesn't match! that way if an
// extra field gets added to the HTML but not the JSON specification, it
// won't get rendered and the error will hopefully be caught by CI before
// it hits production
const template = new Template(contactTemplate, contactTemplateData)
template.render({name: 'Bob', phone: '214-515-6932'})
```

The authoring tool, which would be an HTML component, would make it very
clear what's being replaced, by highlighting it. There would also be a
command-line tool for inspecting it that would print it out on a color
console, so the developers can be careful to make sure all example data
is replaced and won't inadvertantly show up.

