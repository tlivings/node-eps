# Standard Logger

### Author 

Trevor Livingston

### Status

Draft

### Date

10/21/2016

---

# Overview

The proposal is to add a standard logger to node core, such that logging events between applications, core, and various user land
modules logging events can be observable. 

### Problem Statement

There are many good user land logging modules. Howevever, between multiple user land modules there is no consistency with logging, and 
no potential common output destinaton to make observability up and down the stack possible.

For example, there is no way today for an application to configure logging once at the applciation level and be able to capture logs 
from core and various sub modules.

This is in part due to the fact that there is no standard logger in core for modules and applications to use, or to build abstractions
off of.

### Proposed Solution

Add a standard logger to core, which is merely (for all intents and purposes) a timestamped event emitter for objects.

Examples of this concept:

- https://github.com/rvagg/bole
- https://github.com/tlivings/log-emit

Assuming everyone down the stack (including core) were using something like the above, it would be possible to configure log event
listeners once at a centralized location for use with forwarding to a service, logging to an application's logger of choice, etc.
