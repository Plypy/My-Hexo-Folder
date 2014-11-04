title: 'The 10th Week of GSoC'
date: 2014-07-26 10:00:20
tags:
- GSoC2014
- Weekly-Blog
categories:
- GSoC2014
---

### Announcement for Dashboard 2.0
Exciting week!!!

After lots of testing and fixing, I'm proudly annoucing that the new Dashboard has been released. Check it out https://id.openmrs.org

Well, the new Newdashboard hasn't changed so much on the outside, so you may find no much differences. Except for the email selecting part of profile page. Most the chagnges have happened in the backend. We've replaced the data storage to Mongo, for more possibility in the future.

However, I can't say this upgrading is smooth. We've used lots of time on adapting to legacy datas. The new data model is somehow relatively easy to design and implement. But I have to change a lot for backward compatibility, and this process had spent lots of our time, about weeks.

Anyway, briefly summarize.

### Done
1.  Collaborate with Elliott to deploy the 2.0.
2.  Fix some bugs of it.
3.  Checked and discussed the RESTful APIs with Elliott.

### ToDo
1.  RESTful APIs
