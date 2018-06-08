---
layout: post
title: Redux-Form with External Submit Button
date: 2018-06-08 05:22:09.000000000
type: post
published: true
status: publish
excerpt: 
    In this article, I'll show how to combine a redux-form
    with an external submission button ... 
categories:
- JavaScript
- React
- Redux
- Redux-Form
- blog
tags:
- JavaScript
- React
- Redux
- Redux-Form

author:
  login: nikolay.grozev@gmail.com
  email: nikolay.grozev@gmail.com
  display_name: nikolay.grozev@gmail.com
  first_name: 'Nikolay'
  last_name: 'Grozev'
---

<div id='introduction'/>
# Introduction

Recently, I've been working on a [React](https://reactjs.org/)/[Redux](https://redux.js.org/)
app which uses the [redux-form](https://redux-form.com/) library for web forms.
It's a great library which seamlessly integrates all aspects of web forms, including validation, 
into the React/Redux data flow.

Below is a very simple example of a `redux-form` based form:

```javascript
import React, { Component } from 'react';
import { Form, Field, reduxForm } from 'redux-form';
import { TextField } from 'redux-form-material-ui';
import RaisedButton from 'material-ui/RaisedButton';

class SimpleForm extends Component {
  render() {
    const { handleSubmit, valid, submitting, onSubmit } = this.props;

    return (
      <Form onSubmit={handleSubmit(onSubmit)}>
        <Field name="firstName" hintText="First Name" component={TextField} fullWidth />
        <Field name="lastName" hintText="Last Name" component={TextField} fullWidth />
        <RaisedButton label="Submit" type="submit" disabled={valid || submitting} />
      </Form>
    );
  }
}

function validate(form) {
  const errors = {};
  if (!form.firstName) {
    errors.firstName = "required";
  }

  return errors;
}

export default reduxForm({ form: 'simple-form', validate })(SimpleForm);
```

This example uses the [material-ui](https://material-ui.com/) library
and its `redux-form` bindings. We defined 2 text fields (`firstName` and `lastName`)
and a submission button, which is only enabled if the form is valid and is not being submitted.
Finally, we register the form with `Redux` and bind it to a validation function using the
`reduxForm` function.

Behind the scenes, `redux-form` invokes the `validate` function 
on every field change and keeps track of whether the user has touched any field.
This information is then available via the component properties.
You can look at the [official documentation](https://redux-form.com/) of `redux-form`
to see all relevant properties and quite a few examples.

While this is a very simple and intuitive way to build forms, it has one major problem.
It assumes that the form and the submission button are in the same component.
However, in some cases we need to submit a form from outside of its component.
For example, a form can be embedded as a step/screen within a wizard.
Thus, the "NEXT" button of the parent wizard component must submit the embedded forms.

This setup poses 2 problems:

- The button in the parent component can not access the form component properties. Therefore, it can not check if the form is ready to be submitted (e.g. all fields are valid);
- The button in the parent component can not programmatically submit the form;


<div id='solution'/>
# The Solution

After a lot of research and experimentation, I managed to work around these issues.
A form component can observe when it becomes ready to be submitted and signal
the parent component via a specially provided callback

The following form demonstrates this idea. It does not have a submission button and 
accepts a callback function `enabledCallback` via its properties. We use the `componentDidUpdate`
life cycle method to detect when a component becomes (un)available for submission:

```javascript
import React, { Component } from 'react';
import { Form, Field, reduxForm } from 'redux-form';
import { TextField } from 'redux-form-material-ui';

class ChildComponentForm extends Component {
  componentDidUpdate(prevProps) {
    const enabled = (this.props.valid && !this.props.submitting) || !this.props.anyTouched;
    const wasEnabled = (prevProps.valid && !prevProps.submitting) || !prevProps.anyTouched;

    if (enabled !== wasEnabled) {
      // Signal to the parent component that the form can be submitted
      this.props.enabledCallback(enabled);
    }
  }

  render() {
    const { handleSubmit, onSubmit } = this.props;

    // Standard form but no submit button - it's in the parent
    return (
      <Form onSubmit={handleSubmit(onSubmit)}>
        <Field name="firstName" hintText="First Name" component={TextField} fullWidth />
        <Field name="lastName" hintText="Last Name" component={TextField} fullWidth />
      </Form>
    );
  }
}

function validate(form) {
  const errors = {};
  if (!form.firstName) {
    errors.firstName = "required";
  }

  return errors;
}

export default reduxForm({ form: 'child-form', validate })(ChildComponentForm);
```

From the parent component, we can pass a callback function which saves in the state
whether the form can be submitted. Based on the state, we can conditionally
disable the submission button.

While rendering the form in its parent, we preserve a reference to its 
respective component as a member variable.
We can then use this reference to programmatically submit the form when
the submission button is clicked.

The following demonstrates this approach:

```javascript
import React, { Component } from 'react';
import RaisedButton from 'material-ui/RaisedButton';
import ChildComponentForm from './ChildComponentForm';

export default class App extends Component {
  constructor(props) {
    super(props);

    // We'll keep in state whether the submission button is enabled
    this.state = { submitButtonEnabled: true };
  }

  render() {
    // Render the form - pass the callback and obtain a reference
    return (
      <div>
        <ChildComponentForm
          onSubmit={this.onSubmit}
          // While rendering - save a reference to the form
          ref={form => this.formReference = form}
          // Pass a callback for when enabled
          enabledCallback={this.enabledCallback}
        />
        <RaisedButton
          label="Submit"
          disabled={!this.state.submitButtonEnabled}
          onClick={this.onSubmitClick}
        />
      </div>
    );
  }

  // Callback for when the form is submitted
  onSubmit = (form) => alert(JSON.stringify(form, null, 2))
  
  // Callback for the button - use the saved form reference to submit it 
  onSubmitClick = () => this.formReference.submit()

  // Callback for when form can be submitted
  enabledCallback = (enabled) => this.setState(prevState => ({
    ...prevState,
    submitButtonEnabled: enabled
  }))
}
```
