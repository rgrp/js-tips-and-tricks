Javascript tips and tricks I've found useful. Contributions welcome (just
visit the source, make sure you are logged in and hit edit - or fork and
patch).

# Here beginneth the tips

## Deep Copy

**Warning**: full deep copy is **hard**. The following are only appropriate for "simple" objects.

### JQuery

Copy (clone) in javascript with JQuery::

    // Shallow copy
    var newObject = jQuery.extend({}, oldObject);

    // Deep copy
    var newObject = jQuery.extend(true, {}, oldObject);

### Underscore

Note underscore *does* not do a deep copy at all -- it will only do copy on
first level attributes::

    _.extend({}, myobject);

### JSON

Deserialize to and from JSON (warning: not at all performant!)::

    var newobj = JSON.parse(JSON.stringify(oldobj));


## Check if a variable is undefined

    // The typeof operator always returns a string!
    if (typeof(variable) === "undefined") {
      // do something
    }

## Simile Inheritance in Javascript

### John Resig's original

From <http://ejohn.org/blog/simple-javascript-inheritance/>

    /* Simple JavaScript Inheritance
     * By John Resig http://ejohn.org/
     * MIT Licensed.
     */
    // Inspired by base2 and Prototype
    (function(){
      var initializing = false, fnTest = /xyz/.test(function(){xyz;}) ? /\b_super\b/ : /.*/;
      // The base Class implementation (does nothing)
      this.Class = function(){};
      
      // Create a new Class that inherits from this class
      Class.extend = function(prop) {
        var _super = this.prototype;
        
        // Instantiate a base class (but only create the instance,
        // don't run the init constructor)
        initializing = true;
        var prototype = new this();
        initializing = false;
        
        // Copy the properties over onto the new prototype
        for (var name in prop) {
          // Check if we're overwriting an existing function
          prototype[name] = typeof prop[name] == "function" && 
            typeof _super[name] == "function" && fnTest.test(prop[name]) ?
            (function(name, fn){
              return function() {
                var tmp = this._super;
                
                // Add a new ._super() method that is the same method
                // but on the super-class
                this._super = _super[name];
                
                // The method only need to be bound temporarily, so we
                // remove it when we're done executing
                var ret = fn.apply(this, arguments);        
                this._super = tmp;
                
                return ret;
              };
            })(name, prop[name]) :
            prop[name];
        }
        
        // The dummy class constructor
        function Class() {
          // All construction is actually done in the init method
          if ( !initializing && this.init )
            this.init.apply(this, arguments);
        }
        
        // Populate our constructed prototype object
        Class.prototype = prototype;
        
        // Enforce the constructor to be what we expect
        Class.prototype.constructor = Class;

        // And make this class extendable
        Class.extend = arguments.callee;
        
        return Class;
      };
    })();

Usage::

    var Person = Class.extend({
      init: function(isDancing){
        this.dancing = isDancing;
      }
    });
    var Ninja = Person.extend({
      init: function(){
        this._super( false );
      }
    });

    var p = new Person(true);
    p.dancing; // => true

    var n = new Ninja();
    n.dancing; // => false 

### Steffen Rusitchka version

    /* Steffen Rusitchka version
     * <http://www.ruzee.com/blog/2008/12/javascript-inheritance-via-prototypes-and-closures>
     * Creative Commons Attribution licensed
     */
    (function(){
      var isFn = function(fn) { return typeof fn == "function"; };
      PClass = function(){};
      PClass.create = function(proto) {
        var k = function(magic) { // call init only if there's no magic cookie
          if (magic != isFn && isFn(this.init)) this.init.apply(this, arguments);
        };
        k.prototype = new this(isFn); // use our private method as magic cookie
        for (key in proto) (function(fn, sfn){ // create a closure
          k.prototype[key] = !isFn(fn) || !isFn(sfn) ? fn : // add _super method
            function() { this._super = sfn; return fn.apply(this, arguments); };
        })(proto[key], k.prototype[key]);
        k.prototype.constructor = k;
        k.extend = this.extend || this.create;
        return k;
      };
    })();

Usage::

    var X = PClass.create({
      val: 1,
      init: function(cv) { 
        this.cv = cv; 
      },
      test: function() { 
        return [this.val, this.cv].join(", "); 
      }
    });

    var Y = X.extend({
      val: 2,
      init: function(cv, cv2) { 
        this._super(cv); this.cv2 = cv2; 
      },
      test: function() { 
        return [this._super(), this.cv2].join(", "); 
      }
    });

    var x = new X(123);
    var y = new Y(234, 567);

