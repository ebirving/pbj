# Custom Directives

[Screencast](https://youtu.be/_JKtEbOfX0o)

## Lenin's Objections
- DRY a given Angular app by extracting repeating logic and HTML into custom directives
- Explain the purpose of each of the four directive options, and the four options for the "restrict" directive, E, A, C, M
- Use a custom directive to render an array of objects

Note that what we cover in class isn't using Firebase; it's picking up where you left off yesterday afternoon.

```
git checkout -b templating-and-routing-with-comments origin/templating-and-routing-with-comments 
```

## This class is about tidying up

That is: breaking things up into separate files.

We approached Angular a bit differently than we did, say, Node. You see how we have all of our controllers in one file? We could totally split those up into separate files, and in Node, we probably would have started off that way. Here, though we're starting with very few files.

Take a look at the Grumbles controller file. It's only 46 lines long and I feel like it's hard to tell which controller is where. It would only take me an extra 2 seconds to figure out, though, which begs the question...

**Why is 2 seconds important? [Relevant visual aid.](http://i.imgur.com/Ssz6pjF.png)**

### All of programming is shortcuts

Programming is making shortcuts. The first programmer wrote in 0s and 1s. The second programmer got tired of writing 0s and 1s and invented machine code to do it for him (or her). The third programmer got tired of writing machine code and invented assembly code. The fourth programmer got tired of writing assembly and invented C. The fifth programmer got tired of writing C and invented Ruby. And on the sixth day, God rested.

Indenting is a tiny thing that has a tremendous impact on the usability of your code. Breaking things up into files is the same. It doesn't affect the performance of your app at all -- maybe by a couple milliseconds -- it just improves the readability.

##### Turn and talk: What things do you see that aren't DRY in Grumblr?

Our focus is going to be...

## DRYing the HTML

##### What HTML isn't DRY?

The `index` and `show` pages have almost identical HTML, as do the `new` and `edit` pages.

##### Why would we want to DRY the HTML?

Continuity across an app is a big part of UX. If a Grumble appears one way on one page, and another way on another page, that creates confusion for users. To that end, we can copy and paste HTML from one view to another, but when inevitably we want to change that HTML, it means copying and pasting again.

We're going to do this with...

## Custom directives

##### What are some *standard* directives we've seen so far?

We've seen a lot of attributes: `ng-repeat`, `ng-app`, and so on. But directives can also be entire elements. Angular lets you create, say, `<grumble>` and `<comment>`.

##### So what *is* a directive, anyway?

Basically, a directive is some HTML defined by Angular. A directive can be an attribute, an element, a class, or even a comment.

All HTML elements have behaviors: anchors take you to a page when you click on them, textareas let you write stuff inside them, and so on. Angular lets you create your own HTML elements and give them behaviors you define.

Ever wished there was a `<comment-box>` or a `<random-bill-murray-img>` element? Now you can make one.

One of Angular's sort-of "mission statements" is "to be what HTML would have been if it was designed from the start with web apps in mind."

### You can think of directives as "Angular elements"

To help me keep straight what Angular's terms mean, I made a [cheat sheet of which Angular terms map to which Rails terms](../angular-vs-rails.md).

Directives are most like helpers in Rails.

##### What are some Rails helpers you remember?

`form_for`, `link_to`, `render partial`, and so on.

##### What do those helpers do?

They all add HTML to a view.

### "But what about separating my concerns?"

You're discouraged from using the `onclick=` attribute, and now all of a sudden you're being told to use `ng-click=`?

##### Why is this bad?

##### Why is this good?

I can think of a few reasons:

1. We don't have to put event listeners everywhere
- It makes the HTML easier to read, whereas in Backbone templates are sort of strewn about and it's not so easy to see which goes where
- It makes the HTML make *more sense*, somehow. HTML is meant to tell you the function of content, and this lets you be much more specific about that function. It's (theoretically) easier to read than Javascript, and it's more useful than just defining semantics.

### Custom directives are the "flagship" of Angular

Without them, Angular is just another MVC framework.

So: let's make one!

## Grumble `show`

- In `index.html`, add this custom directive: `<my-custom-directive></my-custom-directive>`. Refresh the page, and you shouldn't see anything.

  **Note about self-closing tags:** This directive doesn't have any text content, so you could use a self-closing tag, `<my-custom-directive />`. However, Angular's pretty picky about self-closing tags. If your entire page goes blank when you're using a custom directive with a self-closing tag, try using open and close tags instead.

- Now we'll give the directive its behavior. Let's make `js/directives/grumble.js`.

- Next we'll set up the actual Javascript. Directives look like pretty much every other module:

  ```js
  (function(){
    var directives = angular.module('grumbleDirectives',[]);
    directives.directive('myCustomDirective', function(){
        
    });
  })();
  ```

  One thing to note is that Angular expects you to write the directive's name as camelCase inside the directive *JS*, but as spine case inside the *HTML*. `.directive('myCustomDirective')` automatically turns into `<my-custom-directive>`.

- Now we'll tell the directive what to use as a template:

  ```js
  (function(){
    var directives = angular.module('grumbleDirectives',[]);
    directives.directive('myCustomDirective', function(){
      return {
        template: "<h1>Hi there!</h1>"
      }
    });
  })();
  ```

- Finally, inject `grumbleDirectives` into your `app.js`, the way we did with `grumbleServices` and `grumbleControllers`

- Actually finally, include `<script src="js/directives/grumble.js"></script>` in the app's main `index.html`.

...and that's it! Run it, and see what happens.

## Directive options

Directives can be given a parameter called `link`. It'll automatically be run every time an instance of that directive is created.

For example: 

```js
(function(){
  var directives = angular.module('grumbleDirectives',[]);
  directives.directive('myCustomDirective', function(){
    return {
      template: "<h1>Hi there!</h1>",
      link: function(){
        console.log("hello")
      }
    }
  });
})();
```

Now my console will print `hello` once for every instance of `<my-custom-directive>` on a page. So on the `index` page, if there are 10 Grumbles, and I put `<my-custom-directive>` inside `ng-repeat`, `ng-repeat` will duplicate this directive 10 times, and I should see `hello` 10 times.

Angular actually passes into this `link` function an argument called `scope`. This is an object that's available **both in the directive's JS and the directive's HTML**. So anything I add to it in the JS will be available in the HTML, and vice-versa.

For instance, I'm going to add a property called `myName` to `scope`. That will let me show the value of `myName` in the HTML.

```js
(function(){
  var directives = angular.module('grumbleDirectives',[]);
  directives.directive('myCustomDirective', function(){
    return {
      template: "<h1>Hi there, {{myName}}!</h1>",
      link: function(scope){
        scope.myName = "Slim Shady";
      }
    }
  });
})();
```

### Directive methods

You can add entire methods to scope and make those available in your HTML. I'll make a method that alerts my name:

```js
(function(){
  var directives = angular.module('grumbleDirectives',[]);
  directives.directive('myCustomDirective', function(){
    return {
      template: "<h1>Hi there, {{myName}}!<button ng-click='complimentMe()'>Pay me a compliment</button></h1>",
      link: function(scope){
        scope.myName = "Slim Shady";
        scope.complimentMe = function(){
          alert("You're looking good today!");
        }
      }
    }
  });
})();
```

### Directive collisions

Check out what happens when I have an element called `my-custom-directive` with an attribute called `my-custom-directive`:

```html
<my-custom-directive data-my-custom-directive></my-custom-directive>
```

I get a `$compile:multidir` error, which means Angular's telling me, "Hey, you're trying to apply the same directive twice to one element.

You can fix this by telling Angular whether the *element* is the directive, or the *attribute* is the directive.

## What kind of directive do you want?

By default, Angular makes every custom directive available as both an element and an attribute. It considers these to be the same:

```html
<my-custom-directive></my-custom-directive>
<div my-custom-directive></div>
```

### Restricting your directive type

If you only want your directive to be available as an element, you add `restrict: 'E'` to your directive. This will make angular use the `my-custom-directive` *element* and ignore the `my-custom-directive` attribute. If I add `restrict: 'A'`, it does the opposite.

```js
(function(){
  var directives = angular.module('grumbleDirectives',[]);
  directives.directive('myCustomDirective', function(){
    return {
      template: "<h1>Hi there, {{myName}}!</h1>",
      restrict: "E",
      link: function(scope){
        scope.myName = "Slim Shady";
      }
    }
  });
})();
```

I mentioned before that custom directives can be elements, attibutes, comments, or classes. If you're looking for a mnemonic by which to remember these, use `MACE`: coMment, Attribute, Class, Element.

So `restrict: 'C'` would make this work:

```
<div class="my-custom-directive"></div>
```

You could do `restrict: 'M'` to make your directive availble as a comment. However, comments don't actually render any HTML. For instance:

```
<!-- directive:my-custom-directive -->
```

I still see the `console.log` happening, but that's it.

If you want your directive to be availble as any of the four options, you add `restrict:'MACE'` to your directive, and you can use any combination in between.

##### I mentioned that by default Angular lets you use a custom directive as an element or an attribute. This means the default value of `restrict` is what?

### Do you want your directive to *replace* the HTML that calls it, or just go inside it?

If my directive looks like this:
```js
(function(){
  var directives = angular.module('grumbleDirectives',[]);
  directives.directive('myCustomDirective', function(){
    return {
      template: "<h1>Hi there, {{myName}}!</h1>",
      restrict: "E",
      link: function(scope){
        scope.myName = "Slim Shady";
      }
    }
  });
})();
```

...and my HTML looks like this:
```html
<div>
  <my-custom-directive></my-custom-directive>
</div>
```

...what actually gets rendered in the browser is this:
```html
<div>
  <my-custom-directive><h1>Hi there, Slim Shady!</h1></my-custom-directive>
</div>
```

I can add `replace: true` and that will have my template *replace* the element that calls my directive:

```js
(function(){
  var directives = angular.module('grumbleDirectives',[]);
  directives.directive('myCustomDirective', function(){
    return {
      template: "<h1>Hi there, {{myName}}!</h1>",
      restrict: "E",
      replace: true,
      link: function(scope){
        scope.myName = "Slim Shady";
      }
    }
  });
})();
```

```html
<div>
  <h1>Hi there, Slim Shady!</h1>
</div>
```


## Attributes

So far we've seen a bunch of ways of getting things *out* of the Javascript and *into* the HTML. But how do we get things out of the HTML and into the Javascript?

We do so using attributes:

```html
<div class="grumbles" ng-repeat="grumble in grumblesCtrl.grumbles">
  <my-custom-directive some-attribute="I'm an attribute!"></my-custom-directive>
</div>
```

```js
(function(){
  var directives = angular.module('grumbleDirectives',[]);
  directives.directive('myCustomDirective', function(){
    return {
      template: "<h1>Hi there, {{myName}}! {{someAttribute}}</h1>",
      scope: {
        someAttribute: "@"
      },
      link: function(scope){
        scope.myName = "Slim Shady";
      }
    }
  });
})();
```

We'll get to the `@` in a bit. Note that Angular automatically turned `some-attribute` from spine-case to camelCase.

This is **extremely useful** because it gives you a way of passing data directly into your directive via the attribute.

### A note about attributes and validating your HTML

**Why would you want to validate your HTML in the first place?** Aren't we kind-of-past that? Angular can break really easily when, say, you forget a closing `</div>` tag.

Check out what happens when I run the HTML validator with this code inside it:

```html
<!DOCTYPE html>
<html>
  <head><title>Hi</title></head>
  <body>
    <div my-custom-directive></div>
  </body>
</html>
```

It yells at me about using a non-standard attribute -- one that doesn't come built-in-with HTML. You can "fake out" the validator by putting `data-` in front of the attribute:

`<div data-my-custom-directive></div>`

This doesn't affect the behavior of the attribute at all -- Angular just ignored the `data-`.

This is **good, standard practice** because it makes any custom HTML you created -- which could potentially disrupt other components on a page -- much more visible to other developers.

Similarly, the HTML validator doesn't like custom elements, and you **can't** just add `data-` before them to make them work. `<grumble>` doesn't validate, and neither does `<data-grumble>`. So `replace` makes it easier to keep your HTML validated.

### Writing HTML inside a Javascript file is kind of annoying

Angular lets you put all the HTML inside a completely different file using `templateUrl` instead of `template`.

First, make a file inside the `views/grumbles` folder called `_grumble_show.html`. Rails convention for partials is to put an underscore `_` at the beginning of their file name, so we may as well do that here. Inside it, put:

```html
<h1>Hi there, {{myName}}!</h1>
```

Now, replace `template` in the directive's JS file with `templateUrl` and a link to the `_grumble_show.html` file *relative the main `index.html` file*:

```js
(function(){
  var directives = angular.module('grumbleDirectives',[]);
  directives.directive('myCustomDirective', function(){
    return {
      templateUrl: "views/grumbles/_grumble_show.html",
      replace: true,
      link: function(scope){
        scope.myName = "Slim Shady";
      }
    }
  });
})();
```

## Making a more useful directive

What I'd like to use for my template is the HTML that's used for showing the information about a grumble. That is: the HTML that's identical between `views/index.html` and `views/show.html`. We'll make a directive with this as a template.

For now, we'll leave `show` alone and just get this working in `index`.

Cut and paste into `_grumble_show.html` from `index.html` the HTML that you want to be repeated for each Grumble.

The next step is to make `grumble` available inside the partial. If you look at the partial's HTML, there's a whole lot of `grumble.title` and `grumble.id` and so on. We need to "pass in" that `grumble.`, which we can do with an attribute.

In `index.html`, in the space left by all the HTML you cut out, put:

```html
<grumble-show data-grumble="grumble"></grumble>
```

Now I need to make that grumble available inside the partial's HTML. I do this by setting `scope.grumble` equal to the grumble we passed in via the attributes:

```js
(function(){
  var directives = angular.module('grumbleDirectives',[]);
  directives.directive('grumbleShow', function(){
    return {
      templateUrl: "views/grumbles/_grumble_show.html",
      replace: true,
      scope: {
        grumble: "@"
      }
    }
  });
})();
```

It didn't do anything! If we add `{{grumble}}` to the `index.html` all we get is the word "grumble".

This is because `grumble` is an *object*. The `@` in `scope` is for passing strings. If you want to pass an object, use `=`.

```js
(function(){
  var directives = angular.module('grumbleDirectives',[]);
  directives.directive('grumbleShow', function(){
    return {
      templateUrl: "views/grumbles/_grumble_show.html",
      replace: true,
      scope: {
        grumble: "="
      }
    }
  });
})();
```

`views/index.html` should look like this:

```html
<div>
  <a ng-href="/#/grumbles/new">New Grumble</a>
  <div class="grumbles" ng-repeat="grumble in grumblesCtrl.grumbles">
    <grumble-show data-grumble="grumble"></grumble-show>
  </div>
</div>
```

Run it and see what happens.

We've effectively created a little widget we can use anywhere on our app!

##### What might be good candidates for which to make custom directives "in the wild"?
- Date picker
- Video player
- Trello card

## Let's re-use the directive

The whole point of this was to re-use HTML between `index.html` and `show.html`. So:

In `show.html`, delete the HTML that matches the HTML of the custom directive. The directive will fill in that HTML for us now. In its place, put `<grumble-show data-grumble="grumble"></grumble-show>`.

##### Turn and talk: Why doesn't the grumble show up?

Looking at the `show.html` page, all of the grumble data comes from `grumbleCtrl.grumble`. Looking at the `_grumble.html` page, all of the data comes from just `grumble`.

So just delete that occurence of `grumbleCtrl`:

```html
<grumble-show data-grumble="grumbleCtr.grumble"></grumble-show>
```

## 10 min: Do the same for `edit` and `new`

To start, make a second directive in your Javascript just by copying and pasting the existing one and changing the name to `grumbleSave`:

```js
(function(){
  var directives = angular.module('grumbleDirectives',[]);
  directives.directive('grumbleShow', function(){
    return {
      templateUrl: "views/grumbles/_grumble_show.html",
      replace: true,
      scope: {
        grumble: "="
      }
    }
  });

  directives.directive('grumbleSave', function(){
    return {
      templateUrl: "",
      replace: true,
      scope: {
        grumble: "="
      }
    }
  });
})();
```

Put your partial in a file called `views/grumbles/_grumble_save.html`.

**Note:** Don't worry about getting the "Save" and "New Grumble" buttons to work; just focus on getting the form to show up properly.

**Note:** Your partial shouldn't have any references to controllers in it -- just `grumble`. Delete any references to controllers.

### By the end:

`new.html` should be:

```html
<div>
  <grumble-save data-grumble="{}"></grumble-save>
</div>
```

`edit.html` should be:

```html
<div>
  <grumble-save data-grumble="grumbleCtrl.grumble"></grumble-save>
</div>
```

`_grumble_save.html` should be something like:

```html
<form>
  <input type="text" name="title" ng-model="grumble.title">
  <input type="text" name="authorName" ng-model="grumble.authorName">
  <textarea name="content" ng-model="grumble.content"></textarea>
  <input type="text" name="photoUrl" ng-model="grumble.photoUrl">
  <a ng-href="/#/grumbles/{{grumble.id}}">&larr; Back</a>
  <button ng-click="update()">Save</button>
</form>
```

### What about the buttons?

We either have to have the Back/Save buttons or the "New grumble" button or all three.

Your best bet is to have all three, and then to show or hide particular buttons based on whether the user's on `new` or `edit`.

##### How might you conditionally hide buttons?

Add an attribute called `form-type` to the directive element:

```html
<div>
  <grumble-save data-grumble="grumbleCtrl.grumble" data-form-type="edit"></grumble-save>
</div>
```

...and be able to add it to scope in the directive Javascript:

```js
directives.directive('grumbleSave', function(){
  return {
    templateUrl: "views/grumbles/_grumble_save.html",
    replace: true,
    scope: {
      grumble: "=",
      formType: ""
    }
  }
});
```

##### Should formType be `@` or `=`?

Then, show or hide the buttons based on the value of formType:

```html
<form>
  <input type="text" name="title" ng-model="grumble.title">
  <input type="text" name="authorName" ng-model="grumble.authorName">
  <textarea name="content" ng-model="grumble.content">new grumble content</textarea>
  <input type="text" name="photoUrl" ng-model="grumble.photoUrl">
  <div ng-show="formType == 'edit'">
    <a ng-href="/#/grumbles/{{grumble.id}}">&larr; Back</a>
    <button ng-click="update()">Save</button>
  </div>
  <div ng-hide="formType == 'edit'">
    <button ng-click="create()">New Grumble</button>
  </div>
</form>
```

If you aren't caught up yet, you can checkout the solution code:

```
git checkout -b custom-directives-reused origin/custom-directives-reused
```

## Substituting directives for controllers

The problem now is that clicking on "New Grumble" doesn't do anything.

##### Why doesn't clicking "New Grumble" do anything?

We don't have a `create()` method defined inside the partial.

It *is* defined inside `newGrumbleController`:

```js
this.create = function(){
  Grumble.save(this.grumble, function(grumble) {
    $location.path("/grumbles/" + grumble.id);
  })
}
```

I can remove it from the controller and plunk it right in the directive:

```js
directives.directive('grumbleSave', function(){
  return {
    templateUrl: "views/grumbles/_grumble_save.html",
    replace: true,
    scope: {
      grumble: "=",
      formType: "@"
    },
    link: function(scope){
      this.create = function(){
        Grumble.save(this.grumble, function(grumble) {
          $location.path("/grumbles/" + grumble.id);
        })
      }
    }
  }
});
```

##### Turn and talk: what needs to change for this method to work? There are 2 things.

- `this` needs to be changed to `scope`.
- `$location` and `Grumble` are dependencies that have to be injected.

```js
directives.directive('grumbleSave',[
  "$location",
  "Grumble",
  function($location, Grumble){
    return {
      templateUrl: "views/grumbles/_grumble_save.html",
      replace: true,
      scope: {
        grumble: "=",
        formType: "@"
      },
      link: function(scope){
        scope.create = function(){
          Grumble.save(scope.grumble, function(grumble) {
            $location.path("/grumbles/" + grumble.id);
          });
        }
      }
    }
  }]
);
```

This means I can delete the newGrumbleController altogether. So I'll delete that from `js/controllers/grumbles.js`, and I'll remove it from `routes.js`:

```js
when("/grumbles/new", {
  templateUrl: 'views/grumbles/new.html'
}).
```

...and when I try to save a Grumble, it works!

### The last thing to change is this ugly directive

The directive has way too many brackets.

##### Refactor the directive so there aren't so many nested brackets

One attempt:

```js
directives.directive('grumbleSave',["$location", "Grumble", grumbleSave]);

function grumbleSave($location, Grumble){
  return {
    templateUrl: "views/grumbles/_grumble_save.html",
    replace: true,
    scope: {
      grumble: "=",
      formType: "@"
    },
    link: linkFunction
  }
  function linkFunction(scope){
    scope.create = function(){
      Grumble.save(scope.grumble, function(grumble) {
        $location.path("/grumbles/" + grumble.id);
      });
    }
  }
}
```

Going along in this vein, **we don't need to have controllers at all** for these views.

### Skinny controllers

Angular's all about having skinny controllers, in the same way that Rails likes skinny controllers and fat models. 

My [completed version of this app](https://github.com/ga-dc/grumblr_angular/tree/gh-pages) has a grand total of one controller, used just to load all the Grumbles. Everything else is in directives.

A "soft" rule or guideline for when to use directives or controllers is:

> Controllers should be used when you want to do something to a bunch of instances
> Directives should be used when you want to do something to one particular instance

## We've covered four "options":

- `restrict: "EACM"`
- `replace`
- `template` and `templateUrl`
- `link`

##### What's the *right* way to use them?

There isn't one.

### The story of Robin and Angular

I *love* Angular. It's so useful! It lets me rapidly prototype a single-page app in a much more visual way since things are governed so much by the HTML.

However, until recently I was pretty frustrated with Angular.

At my last job, I was asked to learn Angular to make an app. It *kicked my butt*. It seemed so "draw the rest of the effing owl". I couldn't even find adequate definitions for "controller" and "directive", let alone know when to use which. I was spared from total failure only by the fact that GA came along and poached me.

Going it over again before this class, I ran into the same trouble.

#### In retrospect, this reminds me very much of when I was selling Noteboards.

Folks around the world had been asking me to sell them internationally. I had a vague idea that I needed to pay tariffs or at least fill out some paperwork to do so. I looked for instructions. I *couldn't find them anywhere*.

This seemed so "draw the rest of the effing owl". It was like how to register to sell internationally was a state secret. Being the law-abiding citizen I am, and not wanting to get in trouble, I ended up just not selling them internationally at all.

...until one day, about 6 months in, it hit me:

**There is no owl.**

The absence of resources telling me the rules and restrictions and the "right" thing to do meant there weren't rules and restrictions. All I had to do was mail them, and unless I was selling in excess of some random big number or selling some controller substance, I didn't have to worry about covering my butt. The choices were all my own.

### Angular has fewer owls than you think

Unlike, say, Rails, which was written with pretty explicit rules in mind, Angular is in some ways a much more flexible framework: you use it however you want to use it.

There are lots of choices you have to make *for which there aren't "right" answers*, and that have no bearing on the performance of your app whatsoever.

Once I discovered that Angular is owl-less, I started *loving* it.

##### What are some of these choices that don't have right answers?

- Put methods in directives or controllers?
- Use directives as elements or attributes?
- Use Firebase or AJAX?
- Use `<div data-grumble>` or `<div grumble>`?
- Use a CDN or Bower?
- More or fewer files?
  - We put all of the controllers in one big file, but could totally have put them in separate files. The only reason we didn't is there wasn't a pressing need since this app is pretty small. But if we make lots of custom directives it might be good to make a file for each.
  ```js
  // grumbleDirective.js
  angular.module('grumblr').directive('grumble')...
  ```
  ```js
  // commentDirective.js
  angular.module('grumblr').directive('comment')...
  ```
  ```html
  // index.html
  <script src="directives/grumbleDirective.js"></script>
  <script src="directives/commentDirective.js"></script>
  ```
- Organize your files by the model they relate to, or by the type of file?
  ```
  directives/
    grumble.js
    comment.js
  controllers/
    grumble.js
    comment.js
  ```
  ```
  grumble/
    directives.js
    controllers.js
  comment/
    directive.js
    controllers.js
  ```
- Have a whole bunch of modules, or one?
  - We made a module to hold all of our directives:
  ```js
  (function(){
    var directives = angular.module('grumbleDirectives',[]);
    directives.directive("grumble")...
    directives.directive("somethingElse")...
  })();
  ```
  ...but we could just attach them to the main module we defined in `app.js`:
  ```js
  (function(){
    angular.module('grumblr');
    .directive("grumble")...
    .directive("somethingElse")...
  })();
  ```

There's as much a right answer to these as there is to:
- Hyphens or underscores?
- 2 spaces or 4?
- Tabs or spaces?
- Atom or SublimeText?

**The only answer is to pick the one you like the best and stick with it.**

There *are* conventions that you can adhere to, which would be a good idea for maximum readability:

https://github.com/johnpapa/angular-styleguide

The "final solution" for Grumblr was made trying to adhere more-or-less to these guidelines:

https://github.com/ga-dc/grumblr_angular/tree/firebase-and-directives-with-comments-and-grouped-by-module

BONUS: Take that and make it look pretty!

## Review questions

- What does the mnemonic "MACE" stand for?
  - Comment, Attribute, Class, Element
- What's the difference between `template` and `templateUrl`?
  - `template` uses a string as a template; `templateUrl` uses a whole file
- What's the difference between `@` and `=`?
  - Use `@` for strings and "hard" data, use `=` for objects
- In the Grumblr app, should you have a `directives` folder with `grumbleDirectives.js` and `commentDirectives.js` inside it, or a `grumble` folder with `grumbleDirectives.js` inside it and a `comment` folder with `commentDirectives.js` inside it?
  - It's your choice! But it's becoming more convention to do it the second way and organize files by the model to which they refer, rather than by the type of file.
- If I'm making a "grumble cake" custom directive, should I write it `grumble-cake` in the directive file and `<grumbleCake>` in the HTML, or the other way around?
  - The other way around.
- What's the purpose of the `link` property of a directive?
  - You can define scope variables inside it -- that is, the data that's available inside your custom directive. Putting `scope.name = "Steve"` inside `link` means you can use `{{name}}` inside your directive's template.
