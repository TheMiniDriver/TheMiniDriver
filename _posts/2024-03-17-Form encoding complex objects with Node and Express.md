
I've been writing a small demo app, and wanted to keep everything simple, meaning no SPA / frontend frameworks. So I made an express app, with handlebars. I was trying to do a form post with a complex object, including arrays. I ran into an issue on the syntax of the input names so that the object is properly parsed by express. 

## What Not to Do

Initially, you might be tempted to use dots . to indicate nesting within your form field names, like so:

```html
<input type="hidden" name="user.name" value="John Doe" />
<input type="hidden" name="user.permissions[0]" value="admin" />
```

This approach will not work as expected with `express.urlencoded({extended: true})`. It seems like it works with frameworks like Rails, based on Stackoverflow answers. 

## The Correct Method

To properly indicate nested objects and arrays, use square brackets [] for all levels of nesting. This method works seamlessly with Express's URL-encoded parser when `{extended: true}` is enabled, allowing for accurate parsing of complex data structures.

### Example Form Structure

Here's how you should structure your form inputs:

```html
<input type="hidden" name="users[0][id]" value="123" />
<input type="hidden" name="users[0][name]" value="bob" />
<input type="hidden" name="users[0][surname]" value="banana" />
<input type="hidden" name="users[0][permissions][0][name]" value="admin" />
<input type="hidden" name="users[0][permissions][0][state]" value="on" />
```

### Parsing in Express

With your form correctly structured, Express.js can easily parse nested data as JavaScript objects, provided you have the urlencoded middleware configured like so:

```javascript
const express = require('express');
const app = express();

app.use(express.urlencoded({ extended: true }));
```

This setup allows you to access deeply nested form data directly from req.body in your route handlers, making it much easier to work with complex data submissions.
