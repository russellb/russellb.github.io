---
title: "How-to: Write an Asterisk Module, Part 3"
date: "2008-06-30"
categories: 
  - "asterisk"
---

Greetings fellow developers!

This is part 3 of a series on implementing the basic interfaces to provide features to Asterisk.

[Part 1](http://www.russellbryant.net/blog/index.php/2008/06/19/how-to-write-an-asterisk-module-part-1/) - Basic Module Structure [Part 2](http://www.russellbryant.net/blog/index.php/2008/06/20/how-to-write-an-asterisk-module-part-2/) - CDR Handling Interface

In this section, you will see how to implement an Asterisk CLI command. CLI commands are extremely useful for showing configuration, statistics, and other debugging purposes.

We are going to continue editing the module from part 2, [res\_helloworld2.c](/blog/files/asterisk-module/res_helloworld2.c).

The first step is to include the header file which defines the CLI command interface. `#include "asterisk/cli.h"`

Here is the code for a CLI command, "echo". It simply prints back the first argument to the CLI command. The individual parts of this function will be described later.
```
static char *handle_cli_echo(struct ast_cli_entry *e, int cmd, struct ast_cli_args *a)
{
    switch (cmd) {
    case CLI_INIT:
        e->command = "echo";
        e->usage =
            "Usage: echo <stuff>n"
            "       Print back the first argument.n"
            "Examples:n"
            "       echo foon"
            "       echo "multiple words"n"
            "";
        return NULL;
    case CLI_GENERATE:
        return NULL;
    }

    if (a->argc == e->args) {
        ast_cli(a->fd, "You did not provide an argument to echonn");
        return CLI_SHOWUSAGE;
    }

    ast_cli(a->fd, "%sn", a->argv[1]);

    return CLI_SUCCESS;
}

```

The first line of this CLI handler matches the defined function prototype for CLI command handlers. The ast\_cli\_entry argument includes static information about the CLI command being executed. The cmd argument is set when the CLI command is being invoked for a reason other than normal execution from the CLI, which will be explained more later. The ast\_cli\_args argument contains information specific to this one time execution of the command, like the arguments to the command. `static char *handle_cli_echo(struct ast_cli_entry *e, int cmd, struct ast_cli_args *a)`

The switch statement at the beginning of the function handles when the cmd argument is set to indicate that the function is being called for a reason other than normal execution. There are two cases handled.

- CLI\_INIT. Every CLI command handler is called one time with a cmd of CLI\_INIT. This asks the function to fill in the documentation on what the command is, as well as usage documentation.
- CLI\_GENERATE. This cmd is used for implementing tab completion. CLI commands that provide tab completion of arguments must add code here. Implementing tab completion may be explained in a later tutorial. For now, there are many examples throughout the code.

The next code snippet ensures that at least one argument has been provided to the function. It is worth noting here that the ast\_cli\_entry argument is used to retrieve how many arguments define the command itself and the ast\_cli\_args argument is used to retrieve how many arguments were actually specified at the CLI when executing this command. If they are equal, then simply "echo" was provided. `if (a->argc == e->args) { ast_cli(a->fd, "You did not provide an argument to echonn"); return CLI_SHOWUSAGE; }`

Finally, we print a single argument to the CLI, and return that the execution of the CLI command was successful. `ast_cli(a->fd, "%sn", a->argv[1]); return CLI_SUCCESS;`

The next thing that we have to add is the table of CLI commands included in this module. We will use AST\_CLI\_DEFINE() to add a single entry to this table. AST\_CLI\_DEFINE includes the pointer to the CLI command handling function, as well as a brief summary of what the command does. `static struct ast_cli_entry cli_helloworld[] = { AST_CLI_DEFINE(handle_cli_echo, "Echo to the CLI"), };`

Finally, as discussed in [part 2](http://www.russellbryant.net/blog/index.php/2008/06/20/how-to-write-an-asterisk-module-part-2/), we have to modify load\_module and unload\_module to register and unregister the CLI command table with the Asterisk core.

In unload\_module, we add this: `ast_cli_unregister_multiple(cli_helloworld, ARRAY_LEN(cli_helloworld));`

In load\_module, we add this: `ast_cli_register_multiple(cli_helloworld, ARRAY_LEN(cli_helloworld));`

That's it! Recompile and reinstall the module. Take a look at [res\_helloworld3.c](/blog/files/asterisk-module/res_helloworld3.c), for the completed code.

You can then try it out.
```
*CLI> help echo
Usage: echo 
       Print back the first argument.
Examples:
       echo foo
       echo "multiple words"

*CLI> echo
You did not provide an argument to echo

Usage: echo 
       Print back the first argument.
Examples:
       echo foo
       echo "multiple words"

*CLI> echo foo
foo

*CLI> echo hello world
hello

*CLI> echo "hello world"
hello world


```
