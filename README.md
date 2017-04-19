# Simple Startup

This is a startup script a set of script conventions for simple management
of demo scripts and assets. It does the following:

1. Identifies what components can be started.
2. Provides a simple way to start them up
3. Identifies scripting conventions required to support discovery of components.

## Usage Example

~~~
# ./startup
You must provide a list of components that you want to start up.
Available components: all app datafeed workbook

# ./startup all
starting app... started
starting datafeeed... started
starting workbook...
workbook started at 127.0.0.1:1345
~~~

## Assumptions

- This will be used on modern Linux variants, including RHEL and Debian distribution variants.
- Standard scripting tools such as sed, awk, find, etc.. will be avilable. Any missing tools
  that are needed but missing will be documented here for clarity.
- This scripting format makes no attempt to protect the local system from the scripts being called.
  It is imperative that you only run this tool with scripts that have an appropriate trust level,
  i.e. Do not call unknown scripts on systems that you want to protect.

## Script Locations

All of the components are supported by their own scripts, located in the .startup directory, 
relative to the startup script itself. For example, if you have some demo material in
/tmp/foodemo, then your startup script would be at /tmp/foodemo/startup, and the supporing
component scripts would be in

- /tmp/foodemo/.startup/app
- /tmp/foodemo/.startup/datafeed
- /tmp/foodemo/.startup/workbook

Alternately, the startup script itself may be in the .startup directory, if your wish is
to move it out of the way of casual users.

The current working directory will be set to the parent directory of the .startup component
scripts directory before it is executed.

## Script Ordering

By default, component scripts will execute in lexicographic order. If you want to impose
a strict ordering, then create a file named startup.order in the .startup directory.
If the .startup.order file exists, and there are files in the .startup directory that
are not listed in it, then the startup script will provide an error and exit when it is run.

## Component Script Conventions

These script conventions are fairly normative. For the purposes of uniform and reliable
startup behavior, you should consider these as strict requirements for component scripts.

- Each component script must be set to executable.
- Each component script should be a bash script by default. A simple /bin/sh script is acceptable,
  but bash is presumed to be available on all systems that will use this format.
- Each component script must run the component in the background and then exit while the component
  runs. It is ok to run short-lived options in the foreground, but long-lived processes must not
  block component script completion.
- Each component script should exit with a non-zero exit status if it is unable to successfully start
  the component.
- Each component script should provide any necessary endpoints or status details
  that a user would need to know. For example, a url, or a logging directory.

## Not Component Script Conventions

Component scripts are not required to handle component configuration and shutdown.
This scripting format is meant to make it easy to *startup* a set of components from a clean state
with a simple and known configuration. The complexities of managing reliable component lifecycle
and configuration are out of scope for this format at this time.

