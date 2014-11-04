title: 'The OpenMRS ID Dashboard 2.0'
date: 2014-07-03 21:40:15
tags:
- GSoC2014
categories:
- GSoC2014
---

Finally, after weeks of exploring and attempting, the new dashboard is nearly in place.

The main purpose of Dashboard 2.0 is to provide a extendable user model for OpenMRS ID, so we used MongoDB as the backend database. Hence we can add some free-form data to database, as discussed and explained in these talks.
[My Midterm presentation](https://talk.openmrs.org/t/gsoc-2014-openmrs-id-platform-improvements-midterm-presentation/321)
[Disscussion ](https://talk.openmrs.org/t/api-data-model-design-discussion-free-form-data/282)

### Concern for LDAP
While gaining the new features, we can't leave the older behind. So for backward compatibility, we have to reserve a LDAP server, so those Atlassian Crowd based apps could be happy as usual. 

My original plan was to create a LDAP layer on top of the new user data model via [ldapjs](http://ldapjs.org/). However, after some attempts, I found that its not feasible, or more accurately, efficient.
So I turned to the sync plan. Specificly, we'll remain the OpenLDAP server and sync with it. Although this sync is one-directional, that is, we can only sync the changes from MongoDB side to LDAP.

For data migration, currently I don't have a good idea, because I don't know much about the production. So for quickly putting Dashboard 2.0 into production and test, I choosed a dynamic migration approach. In detail, I've bound a query of LDAP to one query method of Mongo, when I don't find any record in Mongo I'll query in LDAP, and if there is one in LDAP, I'll copy that in Mongo.

### New Procedure for Signup
I'll demonstrate this in one diagram, see.

![Signup Procedure](https://lh4.googleusercontent.com/-fk154LsyjMM/U7WFmJq69gI/AAAAAAAAAwg/z1MnsqJU4X4/w726-h565-no/OpenMRS-ID-Dashboard-Procedure.png "Signup Procedure")

### Something about Data Model
Our user related data models are very simple. We only have user and groups schema.

Except from those basic attributes, 2 things need to be mention exclusively.

##### The `user.extra`
it's in [Mixed](http://mongoosejs.com/docs/api.html#schema-mixed-js) type. So you may treat it as a normal json object.
    
In future, we'll use that to store all kinds of other things other clients put into via our API. See this in detail,
[Disscussion ](https://talk.openmrs.org/t/api-data-model-design-discussion-free-form-data/282)

##### The relation between `Groups` and `Users`
One user may be member of different groups, and each group has different users, so it's a "many to many" relation.

To manage relationship in Mongo*, just store the ObjectId of one doc into another, so you can easily reference each other

But considering the number of groups will be small anyway, so for the sake of simplicity, I've just stored the group names in user docs.

And to easily get all the members of one group, each group will have a `userList` array containing usernames and ObjectIds. Having usernames known, we don't have to really query for users in most cases.

However, Mongo* don't have built-in mechanism to query the array belong to one document. But again, considering we won't have too much users :|, I'll just do a plain O(n) one by one search. No need to use any data structrue and algorthim, hahahahhhhhhh...

That should be it.
