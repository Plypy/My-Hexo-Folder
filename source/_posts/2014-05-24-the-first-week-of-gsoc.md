title: 'The First Week of GSoC'
date: 2014-05-24 15:24:21
tags:
- GSoC2014
- Weekly-Blog
categories:
- GSoC2014
---
So ended the first week of GSoC2014. It's been exciting and funny, though there is some exam pressure on me :/ 


## What I Have Done
***
### Refactoring

I've broken the routing strategies into small pieces of files. Instead of placing all routes in `app.js`, now each router is placed in its own files
. E.G. `get(/login)` and `post(/login)` is placed under `login.js`. Above that there is an `index.js` requires all those files.

Besides these files were put into different modules as well, different module handle different business, and they are independent from each other. I'm trying to keep to make everything a module. Nevertheless some basic router were kept by the main project.

So now the whole are structure like these,
```javascript
app.js                    # requires all modules specified in conf.js, and the router folder
routes
| index.js                # requires router file
| login.js    
| logout.js  
modules
| index.js                # requires lib
| lib                    
| | index.js              # requires routes and controller
| | routes                # organized as the former one
| | | ...
```

### Setteled the disagreement on code style

We now officially adopted the [Felix's](http://nodeguide.com/style.html) as the project's code style standard. Due to some historical problems, there isn't, at least for javascript, a code convention for OpenMRS. So maybe in the future, we'll have our own formal code convention as well :D

For the sake of efficiency. I'm now using [SublimeLinter](https://sublime.wbond.net/packages/SublimeLinter) to detect the potential problems and [JsFormat](https://sublime.wbond.net/packages/JsFormat) to format my code.

However, in the process of refactoring, I had to be very very careful. Cause this project is somehow lack of unit testing. That's to say, another to-do added on the list. Bonjour [Mocha](http://visionmedia.github.io/mocha/).

## What I Will Do
***
I think I'll stick to my old plan, thus:

1. Continue the refactoring work, move different views into their respective modules.
2. Clean-up a step further.
3. Learning the db stuff, starting to make some beta designs.
4. Completing the issue-tracking page.

Be hardworking, and be active on community. 

All for delivering the best work!
