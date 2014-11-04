title: 'The 3rd Week of GSoC'
date: 2014-06-08 01:04:24
tags:
- GSoC2014
- Weekly-Blog
categories:
- GSoC2014
---

Oh the 3rd week has ended, time for sum-up. It's been a really really tough week :(

Those [Mongoose](http://en.wikipedia.org/wiki/Mongoose) living in MongoDB are so hard to catch!!!

Let it be terse and concise.

## What I Have Done

1.  I'd spent tons of my time messing around with MongoDB and Mongoose, to build the new data model for the Dashboard. This hard process really enhanced my understanding of Node and database stuff. This [post](/2014/06/03/dealing-with-unique-index-of-mongoose-and-mongodb/) are one of the fruits.

2.  Found some bugs of the dashboard, see [ID-24](https://issues.openmrs.org/browse/ID-24), [ID-25](https://issues.openmrs.org/browse/ID-25).

3.  Created one [wiki page](https://wiki.openmrs.org/display/projects/New+Data+Model+Design) for the new data model design, and opened a [talk topic](https://talk.openmrs.org/t/new-openmrs-id-ideas/245) to discuss some ideas about the new OpenMRS-ID.
4.  Created a [new-db](https://github.com/Plypy/openmrs-contrib-id) branch for the mongodb development.
5.  Used [mocha](http://visionmedia.github.io/mocha/) as the test frame, and adopted the [BDD style of Chai](http://chaijs.com/api/bdd/). See this [file](https://github.com/Plypy/openmrs-contrib-id/blob/new-db/test/user.save.js) as example.
6.  Gained SSH access to the staging server.
7.  Learned slight css knowledge, now the blockquote will align to left :)


## What I Will Do

1.  Continue working with Mongo*. 
2.  Fix some issues on `master` branch, and avoid making conflicts with the `new-db` branch.
3.  Configure the mongo of the remote.
4.  Add more unit tests.
5.  Try out the ldapjs with Mongo*.

That's it. Time for sleeping...
