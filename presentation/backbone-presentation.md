BACKBONE PRESENTATION
=====================

Introducing backbone.js, no more JavaScript spaghetti code
----------------------------------------------------------

## Meta

*  Introduction
*  About me
*  Overview
*  Why this presentation?

## What is Backbone

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


## Components deep dive


### Model

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


### Collections

#### What

> Collections are ordered sets of models.

In MVC there is no notion of "Collections". Backbone added collections
because... they make sense (in a web context).

We often work on sets of items. We now have an entity to make this easier.


#### Example collections

Representing a concrete resource:

* Users
* Latest articles
* Articles promoted to frontpage
* All pages
* All nodes

Abstract collections

* Controls on a slideshow
* Main menu links
* Pager in search results


#### How to create a collection

Just as with models, best is to subclass `Backbone.Collection` but it's
not required.

    var Coworkers = Backbone.Collection.extend({
  
      model: Person,

      comparator: 'uid'
  
    });

Above, the `model` attribute specifies which type of model it contains.

This allows us to pass a json string and the models will be created
automatically.

    var source = [
      {
        name: "Toon Ketels",
        age: 30,
        profession: "developer",
        bio: "<p>Scrum master, backend Drupal developer.</p>",
        uid: 7544,
        id: 'tk-7544-corl'
      },
      {
        name: "An Katrien",
        age: 23,
        country: "the Netherlands",
        bio: "<p>Works in Germany. Loves to gamble.</p>",
        uid: 3400,
        id: 'ak-3400-corl'
      },
      {
        name: "Jan Bollen",
        age: 30,
        country: "the Netherlands",
        profession: "product owner",
        bio: "<p>Loves working with team.</p>",
        uid: 410,
        id: 'jb-410-corl'
      }
    ];
  
    var coworkers = new Coworkers(source);

This will create an object like this:

    {
      _byId: Object,
      length: 3,
      models: Array[3]
    }

`model` attribure can also be a function to dynamically determine the
type based on some properties.


#### Retrieving models

The are many ways to get models from a collection.

By model's index in this collection:

    coworkers.at(1);

By the model's ID:

    coworkers.get(410)
  
Based on some attributes (will return an array):

    coworkers.where({country: "the Netherlands"});

And many more, see [backbone documentationn](http://backbonejs.org/#Collection).


#### Adding & removing

It's also easy to add and remove items from the collection.

Add a new coworker:

    var noob = new Person({
        name: "Dries Gaastra",
        age: 19,
        country: "the Netherlands",
        profession: "themer",
        bio: "<p>Plays with css and javascript all day long.</p>",
        uid: 9016,
        id: 'dg-9016-corl'
    });
    coworkers.add(noob);

Remove the second coworker:

    var someone = coworkers.at(1);
    coworkers.remove(someone);


#### Sorting

Collections are sorted on the `comparator` attribute.

We can change it and sort the collection again.

    coworkers.comparator = 'name';
    coworkers.sort();

    coworkers.comparator = 'country';
    coworkers.sort();


#### Event aggregation

Just like models, collections emit events (add, remove, sort). What's
really nice is that collections aggregate the events of their models.

Below we listen for `change:age` event on the models of the collection.

    coworkers.on('change:age', function(model, value) {
      // ...
    });
  
To get an idea of all the evens emitted do:

    coworkers.on('all', function() {
      console.log('event emitted');
      console.log(arguments);
    });
  
And see it in action:

    noob.set('age', 45);


#### What problem does it solve

Collections are concerned about managing group of models and keeping
them in order.

They are an endpoint to get to individual models.


#### How does this prevent spaghetti code

* Help organize sets of models into groups
* Provide convenience functions to manage those groups
* Emit events and aggregate events of models. We no longer need pointers
  to all the models.


### Views

#### What

Views are objects which do two things. 1) Displaying data (most of the
time models) and 2) capturing user interaction to trigger code.

They can use templates to actually display html.


#### Example views

* List of blogposts
* A blogpost
* A button to paginate


#### Create view

The simples way to create a view is to pass the `el` to it.

    var view = new Backbone.View({el: $('#main') });

Here we created a standard Backbone view. Each view has an `el` attribute
which refers to the root DOM object created in the browser.


#### Subclass it

Like most of the things, we better subclass the standard Backbone view.

Furthermore, we don't want to pass an existing element but let Backbone
create a DOM element for us.

    var PersonView = Backbone.View.extend({
      className: 'person-view',

      render: function() {
        this.$el.html( '<h2>Toon Ketels</h2>' );

        return this;
      }
    });

    var view = new PersonView();

Backbone will now create a div with the class 'person-view'.

We've also added the render method which will display a name in a h2.

However, it's not in the DOM yet.


#### Adding view to the DOM

The code (object) creating the view should attach it to the dom.

    var view = new PersonView();
    var $container = $('#main');

    $main.append( view.render().el );

What happened here?

`$main.append()` means we're appending something into the `#container`
element.

`view.render()` will add the `<h2>Toon Ketels</h2>` string to the views
element.

`view.render().el` returns root DOM element of the view, so it can be
appended.

The view is displayed in the DOM.


#### Displaying model data

The previous view always displayed the same `<h2>Toon Ketels</h2>`, which
is not very useful.

Pass in a model and render the contents of the model.

    var Personview = Backbone.View.extend({
     'className': 'person-view',
  
      render: function() {
        var title = '<h2>'+this.model.get('name')+'</h2>';
        this.$el.html(title);
  
        return this;
      }

    });

    var toon = new Person({
      name: "Toon Ketels",
      age: 30,
      profession: "developer",
      bio: "<p>Scrum master, backend Drupal developer.</p>",
      uid: 7544,
      id: 'tk-7544-corl'
    });

    var view = new PersonView({model: toon});

Now the view will display the name.


#### Rerender on change

The above works but we run into a problem when some other code changes
the model's attributes.

When toon model's name will be set to "David", the view still displays
"Toon Ketels". We want the view to always display the correct data.

    var Personview = Backbone.View.extend({
      'className': 'person-view',

      initialize: function() {
         this.model.on('change', this.render);
      },
  
      render: function() {
        var content = '<h2>'+this.model.get('name')+'</h2>';
        this.$el.html(content);
  
        return this;
      }

    });

We do so by binding the model's change event to the view's render method.
This is a very common pattern in Backbone.


#### Using templates

This is pretty cool but it's kind of ugly to have HTML strings into our
model. This is especially true if we wanted to also display the bio,
profession, and so... We would have a lot of HTML strings which would be
kind of a mess.

Better would be to separate the HTML out of the view object.

We do so with templates. Underscore comes with a simple templating system
but we can use any JavaScript templating library.

Using templates is a three step process.

1. Create HTML template string somewhere
2. Grab the HTML string and pass it to the template function
3. Compile template with variables into HTML

In our HTML we add the following:

    <script type="text/template" id="person-view">
      <h2><%= name %></h2>
      <p><span class="profession"><%= profession %></span> | <span class="country"><%= country %></span></p>
      <div  class="controls"> <span class='more'>More</span><span class='less hide'>Less</span></div>
      <div class="bio hide"><%= bio %></div>
    </script>

The view will change into:

    var Personview = Backbone.View.extend({
      'className': 'person-view',

      initialize: function() {
        this.model.on('change', this.render);
      },

      template: _.template( $('#person-view').html() ),
  
      render: function() {
        var content = this.template( this.model.toJSON() );
        this.$el.html(content);
  
        return this;
      }

    });

What happens?

    template: _.template( $('#person-view').html() )

We added a property `template`. This will search the DOM for `#person-view`
and grab it's content. This is the template we defined.

The result is passed to our templating function `_.template()` which returns
a function that, once execute swaps the placeholders with the real values.

    var content = this.template( this.model.toJSON() );

Here we actually passed the JSON object of our models attributes to the
template function.

    this.$el.html(content);

The result is set as content of our element. Our entire view looks like this:

    <div class="person-view">
      <h2>Toon Ketels</h2>
      <p><span class="profession">developer</span> | <span class="country">Belgium</span></p>
      <div  class="controls"> <span class='more'>More</span><span class='less hide'>Less</span></div>
      <div class="bio hide"><p>Scrum master, backend Drupal developer.</p></div>
    </div>


#### Catching use interaction

Okay, that was part one of the view's purpose, displaying content. The
other reason views exist is to catch user intent and execute code.

Backbone views have an `events` property to map functions to user interaction.

    var PersonView = Backbone.View.extend({
  
      'className': 'person-view',
  
      initialize: function() {
        this.model.on('change', this.render);
      },
  
      template: _.template( $('#person-view').html() ),
  
      render: function() {
        var content = this.template(this.model.toJSON());
        this.$el.html(content);
  
        return this;
      },
  
      events: {
        'click .more':        'showMore',
        'click .less':        'showLess',
        'click h2':           'goToDetail',
        'mouseover':          'startHighlight',
        'mouseout':           'stopHighlight'
      },
  
      showMore: function(event) {
  
        this.$('.bio').slideDown('medium');
        this.$('.controls span').toggleClass('hide');
      },
  
      showLess: function(event) {
  
        this.$('.bio').slideUp('medium');
        this.$('.controls span').toggleClass('hide');
      },
  
      goToDetail: function(event) {
        window.alert('Go to detail of ' + this.model.get('name'));
      },
  
      startHighlight: function(event) {
        this.$el.addClass('highlight');
      },
  
      stopHighlight: function(event) {
        this.$el.removeClass('highlight');
      }
  
    });

All the functions to the right get executed when the events to the left
are triggered on the view's DOM elements at left.

    events: {
      'click .more':        'showMore',
      'click .less':        'showLess',
      'click h2':           'goToDetail',
      'mouseover':          'startHighlight',
      'mouseout':           'stopHighlight'
    }

All functions change the DOM or trigger alerts.


#### Changing model attributes

Very often, we update the model's attributes with these view events.

    var PersonView = Backbone.View.extend({
  
      'className': 'person-view',
  
      initialize: function() {
        this.model.on('change', this.render);
      },
  
      template: _.template( $('#person-view').html() ),
  
      render: function() {
        var content = this.template(this.model.toJSON());
        this.$el.html(content);
  
        return this;
      },
  
      events: {
        'click h2':           'selectPerson',
        // ...
      },

      selectPerson: function(event) {
        if (this.model.get('selected')) {
          this.model.set('selected', false);
        } else {
          this.model.set('selected', true);
        }
      },
  
      // ...
  
    });

This will trigger an interesting line of events. Let's give an overview:

1. User clicks `h2` title
2. View calls `selectPerson`
3. Function will toggle the `selected` attribute of the model
4. Model emits `change` event
5. Since the view's render function is tied to this event, `render`
   get's called
6. Render takes the model's attributes (some of them were just updated)
   and displays itself with the updated values.

Now, since our view does not display the "selected" state, it's better
not to rerender itself when selected changes. So we need to be more 
specific to what events we listen to.


#### Same model, different view

However, another view is displaying the content of the same model.

The views template:

    <script type="text/template" id="list-view">
      <span class"title" <%- title %> </span> | 
      <span class="select" <%- selected %> </span>
    </script>

We create a collection:

    var Person = Backbone.Model.extend({
      // ...
    });
  
    var source = [
      // ...
    ];
  
  
    var Coworkers = Backbone.Collection.extend({
      // ...
    });
  
    var coworkers = new Coworkers(source);

Our view definition:

    var ListView = Backbone.View.extend({
      tagName: 'li',
  
      className: 'list-view',
  
      initialize: function() {
        this.listenTo(this.model, 'change:selected', this.render);
        this.listenTo(this.model, 'change:name', this.render);
      },
  
      template: _.template( $('#list-view').html() ),
  
      render: function() {
        var content = {
          selected: this.model.get('selected') ? 'selected' : '',
          title: this.model.get('name')
        }
        this.$el.html( this.template( content ) );
  
        return this;
      },
    });
  
    var $list = $('.list');
  
    coworkers.each(function(coworker) {
      var view = new ListView( {model: coworker} );
      $list.append(view.render().el);
    });

We now see 2 interesting things:

* A view is automatically updated by listening for change events.
* A model can have multiple views


#### What problem does it solve

Views are concerned with displaying data and capturing user interaction.


#### How does this prevent spaghetti code?

* User interaction is only captured into view's event hashes
* The only way a user can update models is via views
* The model and its representation is completely separated


### What is the router?




### Syncing with the backend

## All together now

## After thoughts

*  Using backbone: using specific components to augment backend heavy 
   applications vs full javascript client side apps.

*  Backbone: a library or a client side MVC framework?

*  Modular: backbone + requirejs

*  Drawbacks

*  Community and resources
 

## Meta

# Q&A 10 min
