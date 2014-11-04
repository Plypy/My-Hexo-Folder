title: 'Build This Site'
date: 2014-05-14 17:40:35
tags:
---
The reasons I abandon my oringinal ghost blog are the slowness (for poor connection of my university) and the capacitance limitaion of OpenShift.

After messing around with Github Pages and Jekyll for a whole day, I finally found the amazing Hexo.

So here is what I have done to make this site available.

### Jekyll Attempt
First, I used Jekyll to build my site.

But later I found it somehow hard to handle. Here are few reasons,
+ It's hard to switch themes.
+ It's written in Ruby, which I have little knowledge of.
+ Though Jekyll has built-in syntax highlighting, it doesn't support Markdown style codeblocks. And the Github-Pages has limited the gem plugins, so I can't use kramdown with coderay either.
+ I don't like Liquid, and prefer to use mere Markdown for better portability.

Though after some google work, I found Google Code Prettify and make it work with Markdown. But the whole process are just painful to me, I hate it.

### Hexo
So I turned to Hexo.

Like Jekyll, Hexo is a static website generator, but written in Node.js. So it's faster and simpler.

It's much easier to switch themes in Hexo, and I choosed Light theme designed by the Hexo author.

Howerver when I choose the theme, I encountered some problems. Though I changed the theme in `config.yml`, when I `hexo generate`, hexo won't generate new css file. It seems that the `generate` will cache the css file. 

Solutions are simple, run `hexo clean`, and then `hexo generate`. That will clear the cache and generate grand new files for you.


And now, Let's say a colorful _hello_ to the **world**.

~~~c
#include <stdio.h>

int main(int argc, char const *argv[])
{
    printf("Hello world!");
    return 0;
}
~~~

Nevertheless, this site is still under construction. I'll enhance categories and tags functionalities. But I suppose, it should fit my bisic needs currently.
