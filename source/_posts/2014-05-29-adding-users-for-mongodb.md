title: 'Adding Users for Mongodb'
date: 2014-05-29 15:05:12
tags:
- MongoDB
- auth
---
Today after messing around with MongoDB for hours. I finally figured out one thing... adding a user with password for a specifc database of MongoDB. :<

So I better write a blog to note this down...

My situation was simple, there is only one single instance. So I don't need to touch those complex 'replica set' stuff.

So, few things to note:

First enable the auth mechanism, if you use the config file, modify it. By default, config lies in `/etc/mongod.conf`. Uncomment this line,

        auth = true
    
Or you can start `mongod` with `--auth` option.

Then use `mongo admin` to connect to server and switch to the `admin` database, in where you'll create the Admin user, and use this admin user to create a user for our db.

If you only want this user to have the minimum privileges to create other user. You can make its role as `userAdminAnyDatabase`;

However, this role is very limited. So for development convenience, I used `root`. So the commands are,
    
    mongo admin

    db.createUser( {
        user: 'userNameHere',
        pwd: 'passwordHere',
        roles: [
            { role: 'root', db: 'admin'}
        ]
    })

If you got `db.createUser() is not a function`, please check your mongo's version. This method was not introduced until 2.6. And you can use `db.addUser()` in previouse versions.

Then switch to the database you want, and create another user with `readWrite` role. 

    use yourDB

    db.createUser( {
        user: 'userNameHere',
        pwd: 'passwordHere',
        roles: [
            { role: 'readWrite', db: 'yourDB'}
        ]
    })

That's it, by now you can use this user to connect mongo. You can test it by using `db.auth('username', 'password')`

