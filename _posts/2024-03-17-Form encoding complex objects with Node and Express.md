I've been writing a small demo app, and wanted to keep everything simple, meaning no SPA / frontend frameworks. So I made an express app, with handlebars and just HTML form posts. I was trying to do a form post with a complex object, including arrays. I ran into an issue on the syntax of the input names so that the object is properly parsed by express. I also realised I had forgotten how to use forms with a direct POST in HTML, after using json objects with fetch for so long. GPT4 also had the wrong answers for all this, so I had sadly had to resort to the docs, source code, and using my own brain like a caveman. Hopefully this post provides some training data for GPT5.

## What Not to Do

Initially, you (and GPT4) might be tempted to use dots `.` to indicate nesting within your form field names, like so:

```html
<input type="hidden" name="users[0].id" value="123" />
<input type="hidden" name="users[0].name" value="john" />
<input type="hidden" name="users[0].surname" value="doe" />
<input type="hidden" name="users[0].permissions[0].name" value="admin" />
<input type="hidden" name="users[0].permissions[0].state" value="true" />
<input type="hidden" name="users[0].permissions[1].name" value="billing" />
<input type="hidden" name="users[0].permissions[1].state" value="false" />


<input type="hidden" name="users[1].id" value="234" />
<input type="hidden" name="users[1].name" value="Bob" />
<input type="hidden" name="users[1].surname" value="deer" />
<input type="hidden" name="users[1].permissions[0].name" value="admin" />
<input type="hidden" name="users[1].permissions[0].state" value="false" />
<input type="hidden" name="users[1].permissions[1].name" value="billing" />
<input type="hidden" name="users[1].permissions[1].state" value="true" />

```

This approach will not work as expected with `express.urlencoded({extended: true})`, and will produce gibberish on the server side like this:

```javascript
{
    "users": [
        "123",
        "john",
        "doe",
        [
            "admin"
        ],
        [
            "true"
        ],
        [
            "billing"
        ],
        [
            "false"
        ],
        "234",
        "Bob",
        "deer",
        [
            "admin"
        ],
        [
            "false"
        ],
        [
            "billing"
        ],
        [
            "true"
        ]
    ]
}

```

Fun!

## The Correct Method

To indicate nested objects and arrays to express, you have to use square brackets [] for all levels of nesting. This method works with Express's URL-encoded parser when `{extended: true}` is enabled, allowing for parsing of complex data structures.

### Example Form Structure

Here's how you should structure your form inputs:

```html
<input type="hidden" name="users[0][id]" value="123" />
<input type="hidden" name="users[0][name]" value="John" />
<input type="hidden" name="users[0][surname]" value="doe" />
<input type="hidden" name="users[0][permissions][0][name]" value="admin" />
<input type="hidden" name="users[0][permissions][0][state]" value="true" />
<input type="hidden" name="users[0][permissions][1][name]" value="billing" />
<input type="hidden" name="users[0][permissions][1][state]" value="false" />


<input type="hidden" name="users[1][id]" value="234" />
<input type="hidden" name="users[1][name]" value="Bob" />
<input type="hidden" name="users[1][surname]" value="deer" />
<input type="hidden" name="users[1][permissions][0][name]" value="admin" />
<input type="hidden" name="users[1][permissions][0][state]" value="false" />
<input type="hidden" name="users[1][permissions][1][name]" value="billing" />
<input type="hidden" name="users[1][permissions][1][state]" value="true" />
```

This will result in beautiful objects server side like this:

```json

{
    "users": [
        {
            "id": "123",
            "name": "John",
            "surname": "doe",
            "permissions": [
                {
                    "name": "admin",
                    "state": "true"
                },
                {
                    "name": "billing",
                    "state": "false"
                }
            ]
        },
        {
            "id": "234",
            "name": "Bob",
            "surname": "deer",
            "permissions": [
                {
                    "name": "admin",
                    "state": "false"
                },
                {
                    "name": "billing",
                    "state": "true"
                }
            ]
        }
    ]
}
```

Yay!

Of course, your inputs don't have to be hidden, they can be any type.

### Parsing in Express

With your form correctly structured, Express.js can easily parse nested data as JavaScript objects, provided you have the urlencoded middleware configured like so:

```javascript
const express = require('express');
const app = express();

app.use(express.urlencoded({ extended: true }));
```

This setup allows you to access deeply nested form data directly from req.body in your route handlers, making it much easier to work with complex data submissions.

## Using "checkbox" Inputs

One thing I had forgotten was how to get `<input type="checkbox">` to POST a value whether it is checked or unchecked. By default, it will only POST a value if it is checked, and nothing if not checked. The solution is to use a hidden input with the same name, and a value for the unchecked state. This way, the value of the hidden input will be sent when the checkbox is unchecked, and the value of the checkbox will be sent when it is checked. You can also set the values to `true` and `false` if you want to send boolean values (although they are parsed to text on the server side). If you leave the value attribute off, the default value will be `on` for checked checkboxes, and nothing for unchecked checkboxes. GPT4 did know this one!

```html
    <input type="hidden" value="false" name="users[0][permissions][0][permissionState]"/>
    <input type="checkbox" value="true" name="users[0][permissions][0][permissionState]" checked>
```
