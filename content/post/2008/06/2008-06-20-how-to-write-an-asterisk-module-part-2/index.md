---
title: "How-to: Write an Asterisk Module, Part 2"
date: "2008-06-20"
categories: 
  - "asterisk"
---

In [part 1](http://www.russellbryant.net/blog/index.php/2008/06/19/how-to-write-an-asterisk-module-part-1/), I explained the "Hello World" of Asterisk modules. It implemented just enough to properly compile, get loaded into Asterisk, and print something to the Asterisk log when the module was loaded and unloaded. Now, it's time to start making modules that do something useful.

In [res\_helloworld.c](/blog/files/asterisk-module/res_helloworld.c), we implemented a load\_module and unload\_module function. In that module, they just printed to the log. They actually have a much more important purpose in life.

The job of load\_module() is to say, "Hello Asterisk, I'm a module. I can provide some features to Asterisk and here is how you use them."

Conversely, the job of unload module is to say, "Hello Asterisk, again. I'm about to disappear, so please don't use any of these features that I was providing to you anymore. If you do, you'll explode. kthx!"

So, it's time to update this module to provide a feature to Asterisk. We'll start with a CDR (Call Detail Record) handler. It is one of, if not the simplest interface to implement. To keep things simple, we're going to implement an Asterisk CDR handler that once again, just uses the Asterisk logging interface.

The first thing to do is to add the appropriate header file so that module has the appropriate definitions for the CDR interface.

`#include "asterisk/cdr.h"`

Next, we're going to add a new function. This function will get called every time that there is a new CDR to post. It takes a single argument, which is the CDR itself.

` ```
static int cdr_helloworld(struct ast_cdr *cdr)
{
    ast_log(LOG_NOTICE, "We got a CDR for channel '%s'.  "
        "Source: '%s', Dest: '%s', Duration: %ldn",
        cdr->channel, cdr->src, cdr->dst, cdr->duration);

    return 0;
}

``` ` 

Next, as I described before, we have to make the load\_module() and unload\_module() functions aware of this new feature that is implemented by the module.

In load\_module(), we will add a function call to "register" the CDR handler with the Asterisk core. The arguments are the name of the handler, a short description, and the function that the Asterisk core needs to call when a CDR is ready for posting.

`ast_cdr_register("HelloWorld", "Hello World CDR Handler", cdr_helloworld);`

In unload\_module(), we will add a function call to "unregister" the CDR handler form the Asterisk core.

`ast_cdr_unregister("HelloWorld");`

That's it! See the finished code here, [res\_helloworld2.c](/blog/files/asterisk-module/res_helloworld2.c).

Compile it and install it. When you start Asterisk, you should be able to verify that your new CDR handler has been registered with the Asterisk core.

`*CLI> cdr show status ... CDR registered backend: HelloWorld ...`

When you hang up a call, you should see the result of your CDR handler being executed.

`[Jun 20 18:08:29] NOTICE[4922]: res_helloworld.c:36 cdr_helloworld: We got a CDR for channel 'SIP/5001-007e9da8'. Source: '5001', Dest: '586', Duration: 1`

And now, you have implemented an Asterisk module that adds custom handling of CDR records!
