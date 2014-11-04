title: 'The 2nd Week of GSoC'
date: 2014-05-31 19:39:21
tags:
- GSoC2014
- Weekly-Blog
categories:
- GSoC2014
---

The second week has ended, another fun week, yeah!

So let me sum it up.

## What I Have Done

### Refactoring

Well, though in the last week I successfully rearranged the routing logic. When I tried to refactor other source files, I found it's very hard, well almost impossible, to correctly refactor without any mistakes, when there is no tests.

And considering these files will most probably be replaced, so I decided to suspend the refactoring work. So I just simply used JsFormat, one plugin of Sublime Text, to format those files. So they are more readable now. :D

### Fixing ID-12 & ID-22

OpenMRS uses a lot of team-work stuff, like [Wiki](https://wiki.openmrs.org) and [Jira](https://issues.openmrs.org). When people found some bugs or want to add some new functionalities, they can report it on Jira. And here is the categories for [ID-Dashboard](https://issues.openmrs.org/browse/ID).

When I worked on the refactoring, Elliott asked me to create a new issue on it, so we can publicly track my work, and let people comment on that issue. And by that chance I found there are a lot of issues lying there waiting for development.

So I created my refactoring issue, [ID-22](https://issues.openmrs.org/browse/ID-22) and picked up [ID-12](https://issues.openmrs.org/browse/ID-12) and [ID-19](https://issues.openmrs.org/browse/ID-19). Then successfully solved them.

ID-19 is just a simple typo fix, and ID-12 is about the session creating problem.

Elliott found that there are a lot extra sessions were created in database, and it blame to the global-navbar. Because the navbar was a sub module of the dashboard, and when other modules like Wiki want to use it, they could make a `get` request for `dashbordHOST/globalnav` and added it on themselves according to the response.

And then due to faultily designed of Dashboard, it will create session for all requests, whether they are needed or not. Hence, not only the `/globalnav`, but also some thing like `/resource/*` will create sessions. That's terrible, it will add unnecessary pressure to the DB.

And so, I took a look into the sources, and found that, the real problem lies in the way of using `express.session` middleware. The source just directly used `app.use()` for it, this will make express to generate session for all routes. 

After some searching, I found the best practice maybe store the session middleware first, and then use it when necessary. But later I found it will be a huge modification, 'cause there are other middlware depends on session and they are used globally as well. So instead, I simply created a exception list for the session middleware as a temporary fix.

And in the process of solving this issue, Elliott told me that we can create a subApp for those submodules, and then let the main app use it. Like,

```javascript
var subApp = express.createServer();
// don't call subApp.listen
subApp.get('/some', function (req, res, next) {
    // do something    
})
// do something else
app.use('/parentUri', subApp);
```

And now you can visit `/parentUri/some`.

That is a very good feature that could make less coupling. However, note that the subApp works like a middleware. So it will be infulenced by other middleware that the main app used.

### Starting to Dig Mongoose
First, you need to configure MongoDB instance, and I did. The details are [here](/2014/05/29/adding-users-for-mongodb/).

## What I Will Do
1. Continue to configure the db stuff, and make a guide for that.

2. Starting to design the basic data model with Mongoose.

3. Begin to study the unit testing, like Mocha and Jasmine.

3. Fix more issues maybe.


**So that's it, I need to do my laundry :<**

