A new feature landing in AngularJS 1.3 is the ability to bind an expression to a view without adding a watcher and constantly digesting it.

If you've ever worked with large `ng-repeat`'s then you may have come across performance issues attributed to the number of watchers on a view. Bind once aims to go some of the way towards alleviating this by allowing you to decrease the number of expressions being watched, making the digest loop faster and your application more performant.

This is something which has been previously possible with Pasav's [bindonce](https://github.com/Pasvaz/bindonce) library but has proven so sought after that the AngularJS team have ported it into the core AngularJS library.

The syntax is simple, just prepend an expression with `::`. Here's some examples.

A basic expression. 

```language-javascript
<div>{{::item}}</div>
```
An `ng-repeat`. You could also use bind once within the repeat rather than when defining it to only bind once specific expressions.

```language-javascript
<ul>
  <li ng-repeat="item in ::items">{{item}}</li>
</ul>
```

A value passed through to a directive.

```language-javascript
<a-directive name="::item">
```
    
A clever nuance of the feature is the fact that while an expression is undefined the watcher will remain. Once the expression is defined however the watcher will be removed.

Here it is in action with a basic email client app.

<a class="jsbin-embed" href="http://jsbin.com/yufomo/16/embed?output"></a><script src="http://static.jsbin.com/js/embed.js"></script>

Once logged in the email address displayed is bound once as it won't change, if you logout then a watcher will be re-added to the expression. The email subjects are also bound once as they won't change.

As you can see bind once is simple to implement but powerful in execution.
