---
title: "How-to: Write an Asterisk Module, Part 1"
date: "2008-06-19"
categories: 
  - "asterisk"
---

Have you ever wanted to write an Asterisk module? While some of the Asterisk modules are quite complicated, the anatomy of an Asterisk module is pretty simple. Let's start with the "Hello World" of Asterisk modules: res\_helloworld.

This example is based on Asterisk 1.6. However, creating Asterisk modules for Asterisk 1.4 is almost the exact same.

Create a file called res\_helloworld.c in the res/ directory of an Asterisk source tree.

The first thing in every Asterisk module is to include the main Asterisk header file, asterisk.h.

`#include "asterisk.h"`

Next, include the ASTERISK\_FILE\_VERSION macro. This registers the file with the "core show file version" CLI command. This CLI command lists the last SVN revision where that file changed.

`ASTERISK_FILE_VERSION(__FILE__, "$Revision: $")`

Include the Asterisk module header file. This includes the definitions necessary for implementing an Asterisk module.

`#include "asterisk/module.h"`

Let's go ahead and include the header file that lets us use the Asterisk logging interface, as well. This will let us print messages to the Asterisk log so that our new module actually does something.

`#include "asterisk/logger.h"`

It is now time to include the two functions that every Asterisk module must implement. Those are load\_module() and unload\_module(). These functions get called when Asterisk loads and unloads the module.

`static int load_module(void) { ast_log(LOG_NOTICE, "Hello World!n"); return AST_MODULE_LOAD_SUCCESS; }`

`static int unload_module(void) { ast_log(LOG_NOTICE, "Goodbye World!n"); return 0; }`

Finally, every module must include an instance of one of the AST\_MODULE\_INFO macros. This macro includes the necessary code for the module to properly register itself with the Asterisk core when it gets loaded.

`AST_MODULE_INFO_STANDARD(ASTERISK_GPL_KEY, "Hello World");`

The final result should look something like this: [res\_helloworld.c](/blog/files/asterisk-module/res_helloworld.c)

When you run `make` to compile Asterisk, the build system should automatically find your module. It will be compiled and installed with the rest of Asterisk. Compile, install, and run Asterisk. Then, verify that your module has been loaded:

`*CLI> module show like helloworld Module Description Use Count res_helloworld.so Hello World 0 1 modules loaded`

You should also be able to unload and load your module, and see the appropriate message in the Asterisk logger.

`*CLI> module unload res_helloworld.so [Jun 19 10:50:57] NOTICE[26612]: res_helloworld.c:35 unload_module: Goodbye World! *CLI> module load res_helloworld.so [Jun 19 10:51:05] NOTICE[26612]: res_helloworld.c:42 load_module: Hello World! Loaded res_helloworld.so => (Hello World)`

Congratulations! You have successfully written an Asterisk module!

Next, we will start looking at how to start implementing more useful things inside of this module structure.
