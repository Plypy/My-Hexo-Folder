title: 'Difference between "exports" and "module.exports"'
date: 2014-05-27 01:35:55
tags:
- Node.js
- exports
---

In node, if we want to something to be acessible outside, we should export it by assigning it to an attribute of `exports` or `module.exports`.
So we can use it by simply `require`ing it in other files.

But, sometimes we want to use `require` like importing a function. Rookies might write something like these.
```javascript
exports = function() {
    // something
};
```
But this approach will certainly fail. Actually if you want to `export` a function use `module.exports`, instead of `exports`.

The reason lies in the implementation of Node's module. In Node, every source file is a independent module. The `module` object definition is like below, we'll just focus on `exports`.
```javascript
function Module(id, parent) {
    this.id = id;
    this.exports = {};
    this.parent = parent;
    // something else
}
```
And when Node compile your source file, it will wrap it first.
```javascript
(function (exports, require, module, __filename, __dirname) {
  // your codes
})
```
Then node will call this function and pass those parameters to it, so we can use them in our files. The `exports` is the shortcut for `module.exports`. So even if you reassign it a new value, the original object `module.exports` won't be affected at all.

###Conclusion
In a nutshell `exports` is just a reference to `module.exports`, don't assign it a new value. Instead use this.
```javascript
exports = module.exports = function() {
  //blahblah
}
```

