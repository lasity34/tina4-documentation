#  **Securing Form Handling**

<br>

In this tutorial, we will focus on adding security tokens to Tina4 Python forms, specifically CSRF form tokens for POST requests. This will help secure sensitive actions and ensure only legitimate submissions are processed. We’ll build on the form you already have and make it secure, following a step-by-step approach.


This guide will help you to:

<br>

1. Set up formTokens for **CSRF** protection in form submissions.
2. Secure **POST** routes with Tina4 Python’s built-in security.


### Setting Up Your Project for Secure Form Handling


In Tina4 Python, forms require a formToken to protect against CSRF attacks. Tina4 Python handles form validation for you, meaning you do not need to manually validate the formToken in your POST handler. If the token is invalid or expired, the POST route will automatically return an error.

<br>

1. Create the **form.twig** template to include the **formToken**
2. Create a **GET route** to render the form with a **formToken**.
3. Implement a POST route to handle form submissions, relying on Tina4’s automatic token validation.


### Step 1: Updating the Form Template with a formToken

First, let’s create a form template that includes a formToken for CSRF protection. This token helps ensure that the form submission is legitimate and comes from the server that generated the form.

<br>

Here is the **form.twig** template:

<br>

**src/templates/form.twig**

```bash
<!-- src/templates/form.twig -->
{% set title = title if title is defined else "Email Capture Form" %}
{% extends "base.twig" %}

{% block content %}
<h1>{{ title }}</h1>
{% include "navigation.twig" %}
<form method="post" action="/capture">
    <input name="email" type="text" placeholder="Email" required>
    <button type="submit">Send</button>
    <!-- CSRF form token -->
    {{ "capture form" | formToken }}
</form>
{% endblock %}

```

<br>


- **Hidden Form Token (`{{ "capture form" | formToken }}`)**: This directive generates and includes the formToken that protects against CSRF attacks.



### Step 2: Creating a GET Route to Render the Form with a CSRF Token

Now that the form template is created, we need a **GET route** that will generate the **formToken** and render the form. This will help protect the form against CSRF attacks.

<br>

**src/routes/index.py**



```bash
from tina4_python.Router import get, post
from tina4_python.Template import Template
from tina4_python import tina4_auth

# GET route to display the email capture form
@get("/capture")
async def capture_get(request, response):
    # Generate a formToken for CSRF protection
    form_token = tina4_auth.get_token({"formName": "capture"})

    # Render the form.twig template with the formToken
    content = {"token": form_token}
    html = Template.render_twig_template("form.twig", content)
    return response(html)
```

<br>

- **FormToken Generation**: The formToken is generated using `tina4_auth.get_token()` and passed to the form template.
- **Render Form**: The form is rendered with the formToken included, ensuring it is protected against CSRF attacks.

Once you have created the GET route, test it by navigating to `/capture` in your browser. The form should be displayed with the CSRF token included.

<br>

## Step 3: Implementing a POST Route to Handle Form Submission Securely

<br>

In this step, we will create a POST route to handle form submissions for the `/capture` endpoint. Tina4 Python automatically takes care of validating the formToken for us, so we don’t need to manually perform any validation. This means that if the formToken is missing or invalid, Tina4 will deny the request with an appropriate response — specifically, a 403 Forbidden status.


**src/routes/index.py**

<br>

```bash
# POST route to handle form submission
@post("/capture")
async def capture_post(request, response):
    # Extract the email from the form submission
    email = request.body["email"]

    # Simplified success response
    success_message = f"Form submission successful for email: {email}"
    return response(success_message)

```
<br>

**Explanation**:

- When a user submits the form on the `/capture` page, this POST route handles the request.
- **FormToken Validation**: Tina4 Python will automatically validate the formToken. If the formToken is valid, the request proceeds to handle the form data.
- **Form Submission Success**:

    - If the formToken is valid and the form is submitted correctly, the server will process the email field, and a **success message** will be returned: `"Form submission successful for email: {email}"`.
    - This message will be displayed as the response to the user.

- **Form Submission Success**:

    - If the formToken is missing or invalid, Tina4 Python will automatically return a **403 Forbidden** response.
    - This ensures that only legitimate submissions are processed, protecting your application from CSRF (Cross-Site Request Forgery) attacks.

## Summary


To set up secure form handling in Tina4 Python:

<br>

1. **Update the form template** to include a `formToken`.
2. **Create a GET route** to render the form with a `formToken`.
3. **Create a POST route** to securely handle form submissions using Tina4’s built-in form token validation.

By following these steps, your application is protected against common vulnerabilities like CSRF attacks, ensuring secure data submissions.

<br>
