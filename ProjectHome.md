![http://ericnguyen.com/jsload/shared/icon.png](http://ericnguyen.com/jsload/shared/icon.png)

# JSLoad #

## What is JSLoad? ##

JSLoad is a Javascript file loader that we wrote for Instructables. You give it a set of dependencies and groupings, and it loads the files you need, when your executing code needs them. We're releasing it under the LGPL because we're cool like that, and because we'd like to use any enhancements that other folks come up with. You can download the code at the bottom of this blog post, or [check out the test file](http://ericnguyen.com/jsload/doc/) to see it in action.

## Why JSLoad? ##

Generally, we use dependency managers to avoid having to think about all the couplings that exist within our code. A widget we've made may need a whole mess of stuff -- all spread out between different files -- to run. Dependency managers allow us to say, "Give me everything I need for this widget to run," instead of having to figure it out for ourselves, each time, for each widget.

Other, well-written javascript package managers exist (e.g. [jspkg](http://jspkg.sourceforge.net/) and [YUI Loader](http://developer.yahoo.com/yui/yuiloader/)), etc.) So why did we write a new one, and what reasons might you have for using it? In a nutshell, JSLoad is small, flexible, and is designed to work on its own, without the need for any heavyweight framework.

## How does JSLoad work? ##

JSLoad does the basics: you tell it that file 1 depends on file 2, so it loads file 1 first, then file 2. Throw any number of other dependencies into the mix, and JSload figures out the dependency chain and loads the files in the right order.

The real usefulness of JSLoad comes with its ability to group dependencies using tags. Tags are arbitrary labels that you can apply to (i.e. make dependent on) any group of files or other tags. Tags can be applied to single files or multiple files. Multiple tags can be applied to a single file. You can even think of your tags as depending upon a portion of a file (say, class within a file containing several classes.)

As a result, you can mimic most other dependency structures: Chains, trees, or more complicated graphs. You can tag things that often appear together, that share a certain aspect; whatever your usage calls for. At Instructables, for example, we generally have a base set of widgets and features whose dependencies are primarily tree-like. Those little bits are then collected into larger groupings like "editable" or "commentable"; abstract labels that approximate the kinds of interfaces that are common on our site.

Tags are also very useful while refactoring code. Often, because of the flexibility of Javascript, you won't be sure of the best way to split your code across files. Which portions will be used together most often, and should thus be grouped together to reduce HTTP requests? With JSLoad, you can tag the variant groupings, then organize your code as you wish. Your web pages will just call the tags as they need them. Over time, you may find that one tag is used much more often than the others. Using JSLoad, you can refactor your code into a more efficient file structure, without changing any of the script calls in the pages that use the code.

## How is JSLoad used? ##

Here is an example of how to instantiate a new instance of JSLoad:

```
  var jsLoader = new JSLoad(tags);
```


JSLoad instances are intended to be singletons. JSLoad was designed to track state (which files have already been loaded, for example) in one central location.

The "tags" variable passed to the JSLoad instance is a list of tag dependencies. Here is an example:

```
  var tags = [
    { name : "baselib"
    },
    { name : "widget",
      requires : ["baselib"]
    }
  ];  
```

As you can see, "tags" is an array of objects, each defining a tag and its dependencies. In the above example, the "widget" tag depends on "baselib." An implicit part of the tags definition is that, by default (and for conciseness), tags refer to files. So, in the above example, the "baselib" refers to "baselib.js" and "widget" refers to "widget.js".

If a tag doesn't actually refer to a file, but is an arbitrary grouping of your own design, you can set the "tagOnly" property of the tag to "true":

```
  var tags = [
    { name : "baselib"
    },
    { name : "widget",
      requires : ["baselib"]
    },
    { name : "gadget",
      requires : ["baselib"]
    },
    { name : "dostuff",
      requires : ["widget", "gadget"],
      tagOnly: true
    }
  ];  
```

In this case, I've created a tagOnly tag called "dostuff." There isn't any actual file named "dostuff.js." Rather, the tag just indicates that it needs both "widget" and "gadget" (and, by implication, "baselib") to be loaded. All three will thus be loaded in the correct order if I ask for "dostuff."

How do I ask for "dostuff"? Well, somewhere on my page, I might want to do stuff, and thus inline the following Javascript code:

```
  jsLoader.load(["dostuff"], function () {
    var widgie = new Widget();
    var gadgie = new Gadget();    
  });  
```

This tells my JSLoad singleton to run the anonymous function that is the second argument, and to do so as soon as the "dostuff" tag has all of its dependencies taken care of. I can make my load() calls at any point on the page, requiring any combination of tags, and I can repeat them; JSLoad will take care of creating HTTP requests to get the necessary files only once, only when necessary, and in the right order.

You can download the code at the bottom of this blog post, or [out the test file](check.md)(http://ericnguyen.com/jsload/test/test.html) to see it in action. The archive at the bottom of this blog post includes the test file, too.

## Limitations ##

JSLoad has some limitations, due to its implementation. First of all, JSLoad runs asynchronously, to speed up load time on a page and to allow for nested iframes to load script into the top level context. As a result, if you inline dependent script in your page, JSLoad will need to wrap that script to ensure that it isn't executed before the necessary files are loaded. See "How JSLoad is used" above for details.

Second, the list of dependencies you provide to JSLoad needs to be ordered. That means that no file or tag may depend on a file or tag that appears _after_ it. This allows JSLoad to run faster, as it can calculate the dependency tree in one pass, and protects it (and you) from circular dependencies.

Finally, none of your included scripts can do a document.write(). This is generally bad form, regardless, but jsload won't do it's thing in IE if there are any calls to document.write().

We may remove these limitations in future versions, or at least parameterize them so you can decide which side of a trade-off you'd like to take advantage of. In the meantime, enjoy! And, if you have any comments or questions, please use the comment section below.