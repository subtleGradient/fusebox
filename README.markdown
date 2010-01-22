Introduction
------------
Extending JavaScript natives gives you the power to customize the language to fit your needs.
You can add convenience methods like `"hello world".capitalize()` or implement missing functionality like `[1,2,3].indexOf(2)` in JScript. 
The problem is that frameworks / libraries / third-party scripts may overwrite native methods or each other's custom methods resulting in unpredictable outcomes.
Fusebox, a limited<sup><a name="fnref1" href="#fn1">1</a></sup> version of the sandboxing component found in [FuseJS](http://fusejs.com),
avoids these issues by creating sandboxed natives which can be extended without affecting the document natives.

For example:

      var fb = Fusebox();
      fb.Array.prototype.hai = function() {
        return fb.String("Oh hai, we have " + this.length + " items.");
      };
      
      fb.Array(1, 2, 3).hai(); // "Oh hai, we have 3 items."
      typeof window.Array.prototype.hai; // undefined

Screencasts
-----------
Watch the following screencasts for additional information on the history of
sandboxed natives and a full Fusebox code review.

  1. [Sandboxed Natives 101: Screencast One](http://allyoucanleet.com/2010/01/16/sandboxed-natives-one/)
  2. [How to create a sandbox: Screencast Two](http://allyoucanleet.com/2010/01/16/sandboxed-natives-two/)
  3. [How to create a Fusebox: Screencast Three](http://allyoucanleet.com/2010/01/18/sandboxed-natives-three/)
  4. [The Final Countdown: Screencast Four](http://allyoucanleet.com/2010/01/21/sandboxed-natives-four/)

The `new` operator
------------------
      // with `new` operator
      var fb = new Fusebox();
      // or without
      fb = Fusebox();

Supported sandboxed natives
---------------------------
      fb.Array;
      fb.Date;
      fb.Function;
      fb.Number;
      fb.Object;
      fb.RegExp;
      fb.String;

Chainable
---------
      // returns ["a", "b", "c"] <- array & string values are sandboxed
      fb.String("c b a").split(" ").sort();

Working with arrays<sup><a name="fnref2" href="#fn2">2</a></sup>
-------------------
      // like the native Array constructor the sandboxed constructor will return [ , , ]
      var a = fb.Array(3);
      
      // equiv to square-bracket notation [3]
      var b = fb.Array.create(3);
      
      // converting a native array to a sandboxed array
      var c = fb.Array.fromArray([1, 2, 3]);

Working with object instances
-----------------------------
      var a = fb.String("");
      var b = fb.Number(0);
      
      // not falsy like their primitive counterpart
      if (!a) { /* won't get here */ }
      if (!b) { /* won't get here */ }
      
      // a little utility method will smooth things out
      function falsy(value) {
        !value || value == "";
      }
      if (falsy(a)) { /* will get here */ }
      if (falsy(b)) { /* will get here */ }
      
      // will loosely equate to like values
      fb.String("Oh hai") == "Oh hai"
      fb.String("1") == 1;
      fb.Number(1) == 1;
      
      // will *not* strictly equate to like values
      fb.String("Oh hai") !== "Oh hai";
      fb.Number(1) !== 1;
      
      // will *not* equate (loosely or strictly) to other object instances
      fb.String("Oh hai") != fb.String("Oh hai");
      fb.String("Oh hai") !== fb.String("Oh hai");

Converting sandboxed natives to document natives
------------------------------------------------
      var a = fb.String("Oh hai");
      var b = fb.Number(1);
      var c = fb.Array(1, 2, 3);
      
      // results in a document native (primitive)
      "" + a;
      String(a);
      a.valueOf();  
      
      // results in a document native (primitive)
      Number(b);
      b.valueOf();
      +b;
      
      // results in a document native (array object)
      [].slice.call(c, 0);

The `plugin` alias of `prototype`
---------------------------------
      fb.String.plugin.like = function(value) {
        return "" + this == "" + value;
      };
      
      fb.String.plugin.equals = (function() {
        var toString = Object.prototype.toString;
        return function(value) {
          return toString.call(value) === '[object String]' ?
            this.like(value) : false;
        }
      })();
      
      fb.String("1").like(1);   // true
      fb.String("1").equals(1); // false
      fb.String("Oh hai").equals(fb.String("Oh hai")); // true

Gotchas
-------
  - Sandboxed natives don't affect the document natives but
    in some cases document natives may affect sandboxed
    natives. The `__proto__` technique used by Gecko\WebKit
    is affected by modifications to the document natives. To
    avoid this issue simply create the Fusebox before modifying
    the document natives.
    
          // problem
          Array.prototype.sort = function() { return "Oh noes!" };
          var fb = Fusebox();
          fb.Array(3, 2, 1).sort(); // ["O", "h", " ", "n", "o", "e", "s", "!"]
          
          // solution
          var fb = Fusebox();
          Array.prototype.sort = function() { return "Oh noes!" };
          fb.Array(3, 2, 1).sort(); // [1, 2, 3]

  - Sandboxed natives created by the ActiveX / iframe techniques will
    inherit<sup><a name="fnref3" href="#fn3">3</a></sup>
    from the sandboxed Object object's prototype.
    
          fb.Object.prototype.nom = function() {
            return fb.String(this + " nom nom nom!");
          };
          
          fb.String("Cheezburger").nom(); // "Cheezburger nom nom nom!"

Footnotes
---------
  1. The standalone version lacks support for fixing cross-browser `\s`
     RegExp character class inconsistencies and native generics like
     `fb.Array.slice(array, 0)`.
     <a name="fn1" title="Jump back to footnote 1 in the text." href="#fnref1">&#8617;</a>

  2. FuseJS supports additional Array helpers like `fuse.Array.from()` and `fuse.Array.fromNodeList()`.
     <a name="fn2" title="Jump back to footnote 2 in the text." href="#fnref2">&#8617;</a>

  3. The Object object inheritance inconsistency is resolved in FuseJS by assigning `fuse.Object` an object Object
     of a different sandbox instance effectively removing Object object inheritance for the other natives on the `fuse` namespace.
     <a name="fn3" title="Jump back to footnote 3 in the text." href="#fnref3">&#8617;</a>
