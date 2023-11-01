---
title: "pbx_lua: Asterisk Dialplan in Lua"
date: "2007-11-11"
categories: 
  - "asterisk"
tags: 
  - "asterisk-commits"
---

Recently, a new module for writing Asterisk dialplan in the Lua programming language was merged into Asterisk trunk. It was developed by Matt Nicholson of Digium, Inc. See the [commit](http://svn.digium.com/view/asterisk?view=rev&revision=88250) and [mantis issue](http://bugs.digium.com/view.php?id=11140). It will be available in Asterisk 1.6.

From [lua.org](http://www.lua.org/about.html):

> Lua is a powerful, fast, light-weight, embeddable scripting language.
> 
> Lua combines simple procedural syntax with powerful data description constructs based on associative arrays and extensible semantics. Lua is dynamically typed, runs by interpreting bytecode for a register-based virtual machine, and has automatic memory management with incremental garbage collection, making it ideal for configuration, scripting, and rapid prototyping.
> 
> 'Lua' means 'moon' in Portuguese and is pronounced LOO-ah.

This module provides another alternative for native dialplan programming. Check out the [example extensions.lua](http://svn.digium.com/view/asterisk/trunk/configs/extensions.lua.sample?view=markup) file for an example of how it looks.

There has been some discussion about benchmarking pbx\_lua versus the existing extensions.conf and extensions.ael for dialplan execution performance, but no results have been posted as of yet.
