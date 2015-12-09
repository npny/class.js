[Class.JS](https://github.com/npny/class.js)
========
Making javascript easier, 222 bytes at a time.


Usage
-----

```javascript
var Person = new Class
({
  
    initialize: function ( name, email )
    {
        this.name  = name;
        this.email = email;
    },

    sayHi: function ()
    {
        console.log ( "Hi, I'm "  + this.name + " !" );
    }
  
})

var joe = new Person ( "Joe", "joe@gmail.com" );
var bob = new Person ( "Bob", "bob@gmail.com" );

joe.sayHi(); // Hi, I'm Joe !
bob.sayHi(); // Hi, I'm Bob !
```


Advanced usage
--------------

```javascript
var Animal = new Class
({
    initialize: function() { this.weight = 10; }, // initialize() is your constructor
    eat: function() { console.log("Yum !"); }
})


var Cat = new Class(Animal, // That's how you inherit another class
{                    
    // That's how you define "static" variables
    orange: "#ff9900", 
    grey: "#888888",
    
     // You can override the parent constructor with your own if you have some custom initialization to do
    initialize: function() 
    {
        this.weight = 5;
        this.color  = this.grey;
    },

    meow: function() { console.log("Meow !"); },
    lookCute: function() { console.log(":3"); }
});


var Dinosaur = new Class(Animal,
{
    // Or you can just keep the parent constructor
    rawr: function() { console.log("Rawr !"); },
    lookCute: function() { console.log(">:F"); }
});


var Dinocat = new Class(Dinosaur, Cat, // Multiple inheritance works too
{
    initialize: function()
    {
        // You can decide which constructor to call, when, and in what order
        Cat.prototype.initialize.apply(this);
        Dinosaur.prototype.initialize.apply(this);
        // And maybe do some custom initialization
        // And add some more methods
        // [...]
    }                
});

var kitty = new Cat();
kitty.meow(); // Meow !
kitty.weight; // 5

var rex = new Dinosaur();
rex.rawr(); // Rawr !
rex.weight; // 10

var kittyrex = new Dinocat();
kittyrex.eat(); // Yum !
kittyrex.meow(); // Meow !
kittyrex.rawr(); // Rawr !
kittyrex.lookCute(); // :3
kittyrex.weight; // 10

// Since 'Cat' is further down the list than 'Dinosaur', it is inherited last, so Cat.lookCute will overwrite Dinosaur.lookCute. However, Dinosaur's constructor is called after Cat's constructor, so the last value set for weight is 10.

// You can reference static variables from anywhere
Cat.prototype.orange;
Dinocat.prototype.orange;
kittyrex.orange;
```


How it works
------------

In the end, it's not much different from setting up a class the old way :
```javascript
function myClass(a)
{
    this.b = a
}


myClass.prototype.myFunction = function(){};
myClass.prototype.myOtherFunction = function(){};
...
```

It just does the syntactically ugly stuff for you with no real performance impact. And as the title says, once minified the whole thing is only 222 characters long. That's pretty small.

Here's the minified string :

```javascript
function Class(){function _(){return this.initialize.apply(this,arguments)}_.prototype.initialize=function(){};for(var a in arguments){var e=arguments[a].prototype||arguments[a];for(var k in e)_.prototype[k]=e[k]}return _}
```

And here's the human-readable code :

```javascript
function Class()
{
    function _class(){return this.initialize.apply(this, arguments);}
    _class.prototype.initialize = function(){};
    
    for(var a in arguments)
    {
        var extend = arguments[a].prototype || arguments[a];
        for(var k in extend)
            _class.prototype[k] = extend[k];
    }
        
    return _class;
}
```

Let's break it down :

```javascript
function Class()
{
    // We define a new instantiable object, named _class
    function _class()
    {
        return this.initialize.apply(this, arguments); // The constructor forwards everything directly to initialize(). It's a transparent call
    }

    _class.prototype.initialize = function(){}; // We set a default empty function, so that a call to initialize is always valid
    
    // We incorporate each item from the argument list into our _class prototype
    for(var a in arguments)
    {
        // If the current argument is a class, it has a prototype, which is where methods are found, so we inherit that.
        // If it is a plain JSON object, it has no prototype and we simply use the properties.
        var extend = arguments[a].prototype || arguments[a];

        // Add the properties of this item to the prototype
        for(var k in extend)
            _class.prototype[k] = extend[k];

        // If the key already exists, it's overwritten, if not, it's created. That's why the order in which you inherit your classes is important. Rightmost arguments take precedence. Unless you have a good reason, your class definition should be the last argument, so it can override anything it inherits.
    }
        
    return _class; // We then return our _class instantiable object with a complete prototype
}
```

So basically, when you run `var Person = new Class(...)`, Person becomes an instantiable object whose prototype is a merge of every class or object you passed as an argument.


License
-------

class.js is released under the [WTF Public License](http://www.wtfpl.net/txt/copying/).
