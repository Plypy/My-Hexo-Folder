title: 'Dealing With Unique Index of Mongoose and Mongodb'
date: 2014-06-03 20:51:11
tags:
- Mongoose
- MongoDB
- unique-index
---
Mongoose provide a [unique](http://mongoosejs.com/docs/api.html#schematype_SchemaType-unique) attribute for schema types. By setting it as true, you can easily create uniqe index for an attribute.

However there are some issues you must know.

### Mongodb won't ensure one item to be unique from others in the same array.

Say, you want to have a email list for your user schema, and you must ensure all members in the email list are unique from anyone else, whether they are in the same array or not.
So you set this.

~~~javascript
var userSchema = new Schema({
    emailList: {
        type: [String],
        unique: true,
    },
});
var User = Mongoose.model('User', userSchema);
~~~

And now, if you try this
~~~javascript
var user1 = new User({emailList: ['john@doe.com', 'foo@bar.com']});
var user2 = new User({emailList: ['foo@bar.com']});
user1.save();
user2.save();
~~~
You'll get an *E11000* error as desired, everything seems to be fine. But try this,(If you don't get this error please see section below)

~~~javascript
var user3 = new User({emailList: ['john@doe.com', 'john@doe.com']});
user3.save();
~~~
It will pass the unique test... That's definitely not what we want.

The solution is simple, create one your own [validator](http://mongoosejs.com/docs/api.html#schematype_SchemaType-validate). In this situation:
~~~javascript
var chkArrayDuplicate = {
  validator: function (arr) {
    var sorted = arr.slice();
    sorted.sort();

    var i;
    for (i = 1; i < sorted.length; ++i) {
      if (sorted[i] === sorted[i-1]) {
        return false;
      }
    }
    return true;
  },
  msg: 'Some items are duplicate'
};
~~~

And note that, emails are case-insensitive, so you should add a lowercase [setter](http://mongoosejs.com/docs/api.html#schematype_SchemaType-set) as well. Mongoose will apply setter first and then the validator, so that will be enough. It goes like below,
~~~javascript
function arrToLowerCase(arr) {
  arr.forEach(function (str, index, array) {
    array[index] = str.toLowerCase();
  });
  return arr;
}
~~~

###  Sometimes unique index won't work as you desired.

It's normal and often that we'll change our schema during the development & testing. Sometimes when we changed our schema and added the unique qualifier, Mongoose and Mongodb won't reflect our changes. The unique index won't be generated, thus you won't ensure the uniqueness.

Sometimes, that happens when you have stored some duplicates in collections before.

But fix this problem won't be as simple as droping the collections or the databases, or even you restart the `mongod` instance. Because the problem may lie in your codes.

You can first check your collections index, by this command in `mongo` shell, (I'll use previous `user` collections as an example)

    use your_testing_db
    db.users.getIndexes()

If the index was created, then something terribly may have happened. Or most likely, you won't have that index. You can add this listener to your model,

~~~javascript
User.on('index', function (err) {
  if (err) {
    console.error(err);
  }
});
~~~

This listener will listen the `index` event created by `ensureIndex()`. 

>When your application starts up, Mongoose automatically calls ensureIndex for each defined index in your schema.

See [Indexes](http://mongoosejs.com/docs/guide.html#indexes) and [Model.ensureIndexes](http://mongoosejs.com/docs/api.html#model_Model.ensureIndexes).

And keep in mind that, everything in Node.js are asynchronous, so the `ensureIndex()` is as well. So everytime when you tried to fix the problem, you first drop the collection or the database, and the index will also be dropped. And then you run your code again, node fired `ensureIndex()` and before it has done its job, `save()` got fired. So you're messed up as well.

So the best you can do is to keep index created before you write some data to mongo. On production, you'd better create your collections on the database first.

When you test your code, better use another database. And because we need to constantly drop something when testing, you should always listen the `index` event and make sure you write something after that.

### Empty array will be counted

That's a weired feature of Mongodb, if you use unique index, you can't have two documents have empty array. It will be count as duplicates, Even if you use `sparse`. You can check this [SERVER-3934](https://jira.mongodb.org/browse/SERVER-3934).

That's it.
