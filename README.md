# Ember Components and Closure Actions

## Introduction

One of the most powerful features of Ember is its responsiveness. What does that mean? It means that the application is fast and easy to interact with––when a user requests something, via visiting a new page, clicking a button or entering a search query, they not only receive it, but they receive it quickly and seamlessly. 

One of the most common interactions between a user and your basic CRUD app is that of editing a record. Maybe a user is editing their personal profile or the quantity of items in their shopping cart or the description of an item they've posted for sale. In any case, it's easy to imagine a situation in which the person interacting with your app is editing some content that you are presenting to them. 

This kind of interaction can be understood via the concept of "state". If a user is currently editing a record, we consider our app to be in an "editing" state. Once a user submits and saves changes, we are no longer in an "editing" state. 

Ember makes it easy to manage our application's state through the use of components. Components allow us to wrap certain parts of our application, for example a given portion of the view, inside an object. We can give this object properties that represent our application's state and tell this object to respond to certain of the user's actions (for example, the click of a button). 

#### Toggling the Editing State

For the purposes of this example, we'll be working with a basic Ember CRUD app that lists educational resources and allows users to add new resources, edit existing resources and delete resources. In this app we have a `Resource` model and each resource as a title, description, url and topic. 

We'd like our user's interaction with the "edit" functionality to be beautiful and seamless. We want something like this:

![](http://readme-pics.s3.amazonaws.com/flybrary.mov)

In order to implement the above functionality, we'll need to be able to toggle a given resource between "editing" and "not editing" states and be able to accept and save changes made to a resource when it is in the "editing" state. 

We'll build this functionality by creating a component to represent an individual resource and display it in a template. We'll give that component a property to represent the "editing"/"not editing" states and we'll tell our component to respond to the action of a user submitting the edit form. 

## Ember Components

### What is a Component?

An Ember component..

> is a view that is completely isolated. Properties accessed in its templates go to the view object and actions are targeted at the view object. - [Ember docs](http://emberjs.com/api/classes/Ember.Component.html)

For those of us coming from a Rails background, a component is similar to a partial. Just like with a partial, it is a snippet of view-specific code that will display some data to the user. Also just like with a partial, we must pass in any objects we want the component to display when we call our component in the parent template. 

### Why Use Components?

Components make our code re-usable. Instead of re-writing the same code to display a sidebar or a sign-up form, for example, we extract such code into components than can be referenced again and again. 

Components also make our application more responsive. We can define our components to have properties and to respond to actions––such as the click of a button or a double click on a certain element. 

### Defining the Component

We can generate our component with `ember generate component component-name`. This will generate the following:

* `app/components/component-name.js` - This is where we define the properties and actions of our component. 
* `app/templates/components/component-name.js` - This is where we write the handlebars template that constitutes our component. 
* `app/integration/components/my-component-test.js` - Here we write tests for our components. 

Note that all components must be given a dasherized name, like `blog-post` or `side-nav`. Let's generate the following component:

```bash
ember generate component show-resource
```

This component will be responsible for rendering an individual resource on a given template. Our aim is to be able to toggle our component between showing the completed resource and showing the edit resource form. 

Let's write the component's Handlebars template to display a given resource. 

```javascript
// app/templates/components/show-resource.hbs

<h4>{{title}}</h4>  
  <p>{{url}}</p> 
  <p>topic: {{topic}}</p> 
  <p>description: {{description}}</p
```

Now, we can call our component from any template. Let's use our component in the template responsible for rendering an individual resource, our resource show page. 

```javascript
// app/template/resources/resource.hbs

{{component show-resource}}
```

Here, we use the `{{component}}` helper and give it a parameter of the name of the component we want to render. We can also call our component without explicitly using the `component` keyword:

```javascript
{{show-resource}}
```

As it stands, our `show-resource` component would be rendered on template like this:

```
<h4></h4>
<p></p>
<p></p>
<p></p>
```

Oh no! It's blank! Where is our resource? 

Recall that components are completely isolated from the context in which they are being called. So, even though the template in which we are using our component has access to the resource record from the data store, the component doesn't know anything about it. In order to correct this, we need to pass the Ember Data object we are trying to render into the component:

```javascript
{{show-resource resource=model}}
```

Here, we set give our component a property, `resource`, which we set equal to the model object. Now, we need to edit our component template to account for this: 

```javascript
// app/templates/components/show-resource.hbs

<h4>{{resource.title}}</h4>  
  <p>{{resource.url}}</p> 
  <p>topic: {{resource.topic}}</p> 
  <p>description: {{resource.description}}</p
```

Now our component will properly render a given resource's title, url, topic and description. 

Let's move on to the toggle between the component showing a completed resource and showing the form to edit that resource. 

### Toggling State with Component Properties

Before we move on to getting our app to accept and save changes via an edit form, let's discuss how to toggle the appearance of a given resource rendered in our component between a static or complete state and an "editing" state. In other words, how can we allow our user to double click on the resource rendered in the component and see/interact with the edit form?

Recall that the resource represented by our Ember Data object has no concept of whether or not it is being edited. Knowing when to show the completed resource and when to show an edit form is the responsibility of the component. So, we'll give our component a property `isEditing` that defaults to false. When that property is set to `true`, our component template will show the editing form, otherwise it will show the completed resource.
 
#### Setting the Component Property

In `app/components/show-resource.js` we will define our `isEditing` property:

```javascript
// app/components/show-resource.js

export default Ember.Component.extend({
  isEditing: false
});
```

The default value of this property is `false`. 

#### Using the Property in the Component Template

Now, let's edit our component's template to use `if/else` logic to show the edit form when the `isEditing` property is set to true:

```javascript
{{#if isEditing}}

{{input value=resource.title}}
  <p>{{input value=resource.url}}</p> 
  <p>{{input value=resource.topic}}</p> 
  <p>{{textarea value=resource.description}}</p> 
  <button>Save</button>

{{else}}

<h4>{{resource.title}}</h4>  
  <p>{{resource.url}}</p> 
  <p>topic: {{resource.topic}}</p> 
  <p>description: {{resource.description}}</p>

{{/if}}
```

Our component template has direct access to the `isEditing` property since this is a property of the component itself. If `isEditing` has a value of `true`, we display the Handlebars `{{input}}` helpers to show form fields with default values of the title, url, topic and description. Remember that these values are attributes of the component itself. We set them when we called the component in its parent template. 

Otherwise, we display the plain, non-editable title, url, topic and description. 

As it currently stands, however, there is no way for our user to ever see the edit form version of this component. The `isEditing` property has a value of `false` and we have no way to switch, or toggle, it to `true`. Let's fix that now by telling our template to respond to a double click action that has the result of setting the `isEditing` property to `true`. 

#### Creating a Component Action

Let's tell our component to respond to a double click:

```javascript
import Ember from 'ember';

export default Ember.Component.extend({
  doubleClick: function() {
    this.toggleProperty('isEditing');
  },
  isEditing: false;
});
```

Now, when a user double clicks anywhere on the component, i.e. the area of the webpage that shows a given resource, the page will immediately switch to showing the form to edit that resource. 

## Persisting Changes from the Component with Ember Closure Actions

Now we can render a given resource via a component and toggle between editing and not editing that resource when a user double clicks on it. But we can't yet save the changes a user will make to that resource via the edit form. 

This is where we run into some trouble. The component is isolated from the context in which it is called. It has no notion of what template it is being rendered it, it has no access to the data store or awareness of what Ember Data object it received data from. Consequently, we cannot define an action that saves changes to a resource within the component itself. For this, we must define an action in the controller. Then, using an Ember feature called closure actions, we can pass this action *into* our component. 

With closure actions, we can define a particular action in a controller (in this case, the action of saving edits to, or updating, a resource) and pass that action into the component such that the current scope of the action gets passed down to the component as well. Then, we can trigger the invocation of this action directly from the component. 

Let's get started. 

### Defining the Controller Action

At this time, we can only pass actions down from a controller to a component, not from a route to a component. So, we'll define our update action in the Resource Controller:

```javascript
// app/controllers/resources/resource.js

import Ember from 'ember';

export default Ember.Controller.extend({
  actions: {
    update() {
      // coming soon!
    }
  },
});
```

### Passing Down the Action into the Component

Now we're ready to pass our action into the `show-resource` component when we call that component from the template:

```javascript
 {{show-resource saveChanges=(action "update") resource=model}}
```

Here's how this works––the `(action)` helper takes a parameter of the name of the action we are passing into the component, in this case `"update"`. The helper returns the `update` function that we defined in the Resource Controller, wrapped in the current scope. So, we are essentially giving our `show-resource` component an attribute called `saveChanges` and setting it equal to the `update` function we defined in the controller. 

Now, our `show-resource` component has access to the `update` function from the Resource Controller *and* it has access to the scope that the `update` function has access to in the parent template. 

Now we're ready to use the `update` action in our `show-resource` component. 

### Using Closure Actions in a Component

#### Calling the Action

At what point to we want our action to be triggered in the component? When a user hits the "save" button on our edit form. Let's add an action to that button:

```javascript
{{#if isEditing}}
{{input value=title}}
  <p>{{input value=url}}</p> 
  <p>{{input value=topic}}</p> 
  <p>{{textarea value=description}}</p> 
  <button {{action 'submit'}}>Save</button>

...
```
However, if we start up our server and try to save that edit form now, we'll see the following error in the console:

```
Uncaught Error: <resource-library@component:show-resource::ember474> had no action handler for: submit
``` 

#### Handling the Action in the Component

In our component, we need to define an `submit` action:

```javascript
import Ember from 'ember';

export default Ember.Component.extend({
  doubleClick: function() {
    this.toggleProperty('isEditing');
  },
  actions: {
    submit() {
      this.toggleProperty('isEditing');
      this.attrs.saveChanges(this.get("resource"));
    } 
  },
  isEditing: false
});
```

Here, we define our action and give it two behaviors:

* First, toggle our `isEditing` property back to false. This will replace our edit form with the completed resource. 
* Secondly, invoke the `saveChanges` function via

```javascript
this.attrs.saveChanges
```

This will have the effect of invoking the `update` action in turn. Let's take a closer look at our invocation of the closure action.

#### Triggering the Closure Action

Why did we call `this.attrs` in order to invoke the `saveChanges` action? First, lets answer the question: what is `this.attrs`? Well, if we pop a debugger in this component action and use the browser's console to grab `this.attrs`, we'll see:

```javascript
> this.attrs
Object {resource: Object}
resource: Object
saveChanges: ()
__proto__: Object
```

`this` is the component itself and `.attrs` are the attributes of `resource` and `saveChangs` that we assigned to our component when we called it in the parent template. `resource` is set to an object (the model object that was passed in when we rendered the component) and `saveChanges` is set equal to a function.

So, to invoke the `saveChanges` function, we must grab it first from `this.attrs`. 

Notice too that we are passing the value of `this.resource`, our model object, as an argument to the `saveChanges` function. Why is this? 

Our component's `saveChanges` action needs to trigger the controller action: update. However, the update controller action needs to be given access to the object that we are trying to update, our model object.

Recall that we set a property on our component, resource, and set it equal to a model object. So, inside the `submit` action that we defined, we have access to the `resource` property. When the `saveChanges` action triggers the `update` action, it needs to pass `resource` in as an argument to the `update` function. This way, the `update` action will have access to the object whose changes we want to save.

So, we need to build out the `update` action in our controller:

```javascript
import Ember from 'ember';

export default Ember.Controller.extend({

  actions: {
    update: function(resourceToSave) {
      resourceToSave.save();
    }
  }
});
```

Now, our `update` action takes in an argument. When we trigger the `update` action by invoking the `saveChanges` action of the component, we pass in the model object whose changes we want to save by grabbing it from the component's `resource` property.

```javascript
// app/components/show-resource.js

...
this.attrs.saveChanges(this.get("resource"));
...
```

## Conclusion

We saw how to:

* Build and render a component. 
* Use closure actions to handle actions in our component. 

Closure actions allow us to define actions and use them across components. They make it easy to give your normally isolated component access to the context of it's parent template (or templates). They also allow for actions to be passed down through a series of nested components. 

For further reading on Ember's closure actions, check out [this great post](http://alexdiliberto.com/posts/ember-closure-actions/) by Alex DiLiberto.

<p data-visibility='hidden'>View <a href='https://learn.co/lessons/ember-closure-actions' title='Ember Components and Closure Actions'>Ember Components and Closure Actions</a> on Learn.co and start learning to code for free.</p>

<p data-visibility='hidden'>View <a href='https://learn.co/lessons/ember-closure-actions'>Components and Closure Actions</a> on Learn.co and start learning to code for free.</p>
