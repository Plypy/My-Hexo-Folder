title: 'The 9th Week of GSoC'
date: 2014-07-20 13:53:10
tags:
- GSoC2014
- Weekly-Blog
categories:
- GSoC2014
---

### Deployment

Oh, another week of delay... Deployment is somehow exciting and painful.

In this week Elliott and I have tried to do a trial deployment on the staging server. However, we encountered lots of problems that delayed our schedule. Like duplicate emails we found earlier, and also updating other functional modules to adapt the new Dashboard.

I updated the [Migrator](https://github.com/Plypy/OpenMRS-ID-Migrator), and added a verifier for verifing the correctness of the migration, then made PRs for globalnavbar and sso module. And left oauth and groups module for Elliott to work on, sorry for that, but I really don't know how to test these two...

And after we tested the migrator separately on our own machine. Things should be fine now, we think... However, you know, another issue poped out. Last time, due to the famous heartbleed bug of OpenSSL, our people made a global password reset for all acount. Specifically, setting all accounts password as empty, as OpenLDAP won't authenticate a user with empty password. And that is definitely a thing that I have never thought about :/

Not only this, when I was checking this issue, I found that I used wrong password hashing algo, SHA rather than SSHA. And this is because of the different `slapd.conf` between production server and dev machine.

So we created [ID-45](https://issues.openmrs.org/browse/ID-45) and [ID-46](https://issues.openmrs.org/browse/ID-46).


Anyhow, **Summarize**!!

### What I Have Done

1.  Updated Migrator.
2.  Collaborated with Elliott to do a trial deployment of Dashboard 2.0 and solved lots of problems.

### What I Will Do

1.  Finish the deployment.
2.  Learn RESTful APIs


That's it!
