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
tags:
- JavaScript
- React
- Redux
- Redux-Form
---

<div id='introduction'/>
# Introduction

Recently, I've been working on a [React](https://reactjs.org/)/[Redux](https://redux.js.org/)
app which uses the [redux-form](https://redux-form.com/) library for web forms.
It's a great library which seamlessly integrates all aspects of web forms, including validation, 
into the React/Redux data flow.

Below is a very simple example of a `redux-form` based form:

```jsx
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

<div class="mid-page-ads in-body-ads ad-secion">
    <div class="ad-header ad-header-body">Related Topics</div>
    <script id="mNCC" language="javascript">
        if (window.innerWidth >= 1024) {
          medianet_width = "600";
          medianet_height = "250";
          medianet_crid = "459711728";
        } else {
          medianet_width=Math.min(250, window.innerWidth).toString();
          medianet_height = "250";
          medianet_crid = "318234500";
        }
        medianet_versionId = "3111299"; 
      </script>
    <script src="//contextual.media.net/nmedianet.js?cid=8CU4WBM36"></script>
</div>

<div id='solution1'/>
# Solution 1

A form component can observe when it becomes ready to be submitted and signal
the parent component via a specially provided callback

The following form demonstrates this idea. It does not have a submission button and 
accepts a callback function `enabledCallback` via its properties. We use the `componentDidUpdate`
life cycle method to detect when a component becomes (un)available for submission:

```jsx
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

```jsx
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

<div id='solution2'/>
# Solution 2

Another approach is to use the global functions and selectors provided by `redux-form`.
They vary from version to version - in the example below we'll be using version 7.2.0.

Every `redux-form` has a unique name, which is used as an identifier in redux.
In the example below, we define a simple form and associate it with a name:

```jsx
import React, { Component } from 'react';
import { Form, Field, reduxForm } from 'redux-form';
import { TextField } from 'redux-form-material-ui';

class ChildComponentForm extends Component {
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

// The global name for this form will be 'child-form'
export default reduxForm({ form: 'child-form', validate })(ChildComponentForm);
```

This form is agnostic of its parent which will perform its submission and inspect its validity,
as in this example usage:

```jsx
import React, { Component } from 'react';
import { connect } from 'react-redux';
import RaisedButton from 'material-ui/RaisedButton';
import ChildComponentForm from './ChildComponentForm';
import { bindActionCreators } from 'redux';
import { isValid, isSubmitting, submit } from 'redux-form';

// A simple component which uses the form
export class App extends Component {
  render() {
    return (
      <div>
        <ChildComponentForm onSubmit={this.onSubmit} />
        <RaisedButton
          label="Submit"
          disabled={!this.props.formEnabled}
          onClick={this.props.submitForm}
        />
      </div>
    );
  }

  // Callback for when the form is submitted
  onSubmit = (form) => alert(JSON.stringify(form, null, 2))
}

// Use redux-form selectors to check the form's state - e.g. valid, submitting
function mapStateToProps(state) {
  return {
    formEnabled: isValid('child-form')(state) && !isSubmitting('child-form')(state) 
  };
}

function mapDispatchToProps(dispatch) {
  // Bind an action, which submit the form by its name
  return bindActionCreators({
    submitForm: () => submit('child-form')
  }, dispatch);
}

export default connect(mapStateToProps, mapDispatchToProps)(App);
```


