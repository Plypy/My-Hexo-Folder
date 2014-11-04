title: 'The 4th Week of GSoC'
date: 2014-06-14 19:56:26
tags:
- GSoC2014
- Weekly-Blog
categories:
- GSoC2014
---
Let's sum up the 4th week of GSoC!!! 

First, another hard one, I've used a lot of time to fix bug of some async logic. :/

### What I Have Done

1.  Roughly finished the work of integrating the new Mongo* data model with the signup module. Now you can create new account in Mongo*, but you can't log in yet...

2.  Fixed the [ID-6](https://issues.openmrs.org/browse/ID-6). Small defect about the login hint message, easy to fix, just adding a link will be fine. But I think there is still some room for improvement. Maybe when user filled out the form, they can be redirected to the original page. Or simply make it default to open a new page for that link, since there is still the verification work.

2.  Fixed the [ID-14](https://issues.openmrs.org/browse/ID-14), which is about the verification email. The old dashboard will sent the verification email, even if the account isn't got created. This can be easily solved by using [async][1] library. Just simply using [async.series()](https://github.com/caolan/async#seriestasks-callback) will do the trick.

3.  Reimplemented the validation middleware for signup module. Again, by using [async][1], the whole control flow is clearer now. I've spent most of my time on that, because I made some rookie mistakes on writing async code in Node.js.

4.  Created and fixed the [ID-29](https://issues.openmrs.org/browse/ID-29). This issue is about the reuse of typed values of validation form. If the user typed some wrong values, the old dashboard won't cache them, so user have to type them again. Small bugs, but annoying.

### What I Will Do

1.  Complet the work of signup module, and march on auth(login logout) and profile module.

2.  Continue improve the validation code.

3.  Fix some more issues.

2.  Test the Mongoose to see whether is possible for users to change the schema when the server is running. And whether that is a good idea... For details you can see this [talk](https://talk.openmrs.org/t/api-data-model-design-discussion-free-form-data/282) posted by Elliott.

That's it.

[1]: https://github.com/caolan/async
