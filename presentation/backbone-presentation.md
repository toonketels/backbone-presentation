BACKBONE PRESENTATION
=====================

Introducing backbone.js, no more JavaScript spaghetti code
----------------------------------------------------------

## Meta 5

*  Introduction
*  About me
*  Overview
*  Why this presentation?

## What is Backbone 5

Goal:
*  Get a feel for backbone
*  Understand what can be done with it
*  Know the different players
*  Know what it takes to use it

### Backbone

*  Client side mvc
*  like the others...

### What problem tries backbone to solve?

### Components overview

*  views
*  models
*  collections
*  router

### What can we build with this?

*  from easy to hard
*  show of example applications
   *  Brightcove?
   *  From backbone.js

### How to get started?

*  dependencies (_, $)
*  add backbone script tag
*  show
*  Use the objects right away or extend them


## Components deep dive 25

### What are views? 5

*  What is it
*  What problem does it solves
*  Use cases standalone
*  Benefits
*  How does this prevent spaghetti code?
*  Code

### Model 5


#### What

> Models are the heart of any JavaScript application, containing the 
> interactive data as well as a large part of the logic surrounding it: 
> conversions, validations, computed properties, and access control.
>    [Backbone documentation: model](http://backbonejs.org/#Model)

A object to group data and functions related to a data item together.

It provides default events and utility functions to save/get 
data from a data store.


#### Creating a model

To create a standard model, just pass attributes to it.

    var toon = new Backbone.Model({
      name: "Toon Ketels",
      age: 30,
      profession: "developer",
      bio: "<p>Scrum master, backend Drupal developer.</p>",
      uid: 7544
    });

This will create the following JavaScript object.

    {
      _changing: false,
      _pending: false,
      _previousAttributes: {/*...*/},
      _attributes: {/*...*/},
      changed: {/*...*/},
      cid: "cid"
    };

We see there's already a lot more going than just a "bag of data".


#### Common functionality

##### Get properties

The data we've passed are added to the `_attributes` attribute. The
following methods help in getting the values.

    // Get properties
    toon.has('age');
    toon.get('age');
    toon.escape('bio');

Results in:

    true
    30
    &lt;p&gt;Scrum master, backend Drupal developer.&lt;&#x2F;p&gt;


##### Set properties

Besides during initializations, we can set new or existing properties.

    toon.set('age', 31);

The cool thing is we still can access the previous values. Useful for 
comparing.

    toon.changed;
    toon.previousAttributes();

Results in:

    {age: 31}
    {
      name: "Toon Ketels", 
      age: 30, 
      profession: "developer", 
      bio: "<p>Scrum master, 
      backend Drupal developer.</p>", 
      uid: 7544
    }


##### Emitting events

Now, the real power is in emitting events. Whenever a property changes,
the model will emit a `change` event.

Other objects can listen to these events. This is how we can build
loosely coupled systems.

    toon.on('change', function(model) {
      console.log('changed');
      console.log(model);
    });

The above code will be triggered anytime a model attribute changes.

We can also be more specific in the change events we want to listen to.

    toon.on('change:age', function(model, val){
      console.log('changed age');
      console.log(model);
      console.log(val);
    });

Now, the code will only be triggered whenever the `age` attribute changes.

Only when it changes. 

The nice thing is we can set a model attribute multiple times, but the
event is only triggered when the value has changed.


#### Subclassing

Using standard backbone models is good for simple models for which we 
only want to use the basic functionality.

A better approach is the create your own type by subclassing it. This
is the standard way.

    var Person = Backbone.Model.extend({
  
      // Custom methods
  
    });
    
    var toon = new Person({
      name: "Toon Ketels",
      age: 30,
      profession: "developer",
      bio: "<p>Scrum master, backend Drupal developer.</p>",
      uid: 7544,
      id: 'tk-7544-corl'
    });

[Underscore](http://underscorejs.org/#extend) offers the `extend` method for subclassing.

    _.extend(destination, *sources)

Now we can add functionality to the `Person` object and all persons
will have that functionality.


#### What would we add?

What kind of methods and properties would we add?


##### Code run on object construction

The `initialize` method will run on object initialization.

    var Person = Backbone.Model.extend({

      initialize: function() {
        console.log('I run on startup');
      }

    }


##### Default attributes

The `defaults` attribute contains... the attributes the object has 
by default.

    var Person = Backbone.Model.extend({
  
      defaults: {
        country: "Belgium",
        profession: "maid"
      }

    });

When creating the toon instance the values will be:

    {
      age: 30
      bio: "<p>Scrum master, backend Drupal developer.</p>"
      country: "Belgium"
      id: "tk-7544-corl"
      name: "Toon Ketels"
      profession: "developer"
      uid: 7544
    }

Countries is added, profession stays "developer".

Watch out for pointing to objects in the defaults hash.


##### Validation

As we can set properties, we should be able to validate them.

To validate an attribute, check its value in `validate` and return 
something something if incorrect.

    var Person = Backbone.Model.extend({
    
      validate: function(attributes, options) {
        if (typeof attributes.age !== "number") {
          return "Age should be a number";
        }
      }
  
    });

By default, `isValid` is only called just before `save`. But we can 
call it ourself.

    toon.set('age', 'none');
    toon.isValid();               // false

To get the error message we do:

    toon.validationError;         // "Age should be a number"

But we can validate when attributes are set too.

    toon.set('age', 'none', {validate: true});


#### ID's

Backbone will add a "client ID" for each model. It's guaranteed to be
unique for all models over all types for the given client.

    toon.cid                     // c1 

Most of the time, models have ID's on the server to. If ID is an attribute,
Backbone provides a shortcut to access it.

    toon.get('id')               // tk-7544-corl
    toon.id                      // tk-7544-corl

Since not all IDS are called `id`, we can specifically set which attribute
is the server side ID.

    var Person = Backbone.Model.extend({
  
      idAttribute: 'uid',
  
    });

Above we say `uid` is the server side unique identifier.

    toon.get('id')               // tk-7544-corl
    toon.get('uid')              // 7544
    toon.id                      // 7544


#### URL

The cool thing with Backbone is that it allows us the update the models
server side on `save`. This will perform a put/post request to the server.

Therefore ach model can have a url property. It is automatically generated
as urlRoot/[id] and the root will come from the collection.


#### What problem does it solve?

It helps you divide your application functionality into different parts.
Each part (object) has a well defined goal.

And it offers an object to represent your server side data on the client.


#### How does this prevent spaghetti code

* It facilitates clear separation of functionality into different models.
* Tie some functions related to the data to the data (validation, setting,
  getting, defaults, ids).
* It emit events when attributes change
* We don't need to write code to update the resources on the server anymore.


### What are collections? 5
### What is the router? 5
### Syncing with the backend 5

## All together now? 5

## After thoughts 5

*  Using backbone: using specific components to augment backend heavy 
   applications vs full javascript client side apps.

*  Backbone: a library or a client side MVC framework?

*  Modular: backbone + requirejs

*  Drawbacks

*  Community and resources
 

## Meta

# Q&A 10 min
