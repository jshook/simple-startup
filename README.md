# Simple Startup

This is a startup script a set of script conventions for simple management
of demo scripts and assets. It does the following:

1. Identifies what components can be started.
2. Provides a simple way to start them up
3. Identifies scripting conventions required to support discovery of components.

## Usage Examples

~~~
[mock-example]$ ./startup 
INFO: You must select from the following available components, or specify 'all':
app
datafeed
workbook
~~~

~~~
[mock-example]$ ./startup all
# INFO: selecting all components
# INFO: executing: app
app started up
# INFO: executing: datafeed
datafeed started up
# INFO: executing: workbook
workbook started up at http://localhost:3023/testing/
~~~

~~~
[mock-example]$ ./startup app workbook
# INFO: selecting specific components: app workbook
# INFO: component 'datafeed' is being skipped
# INFO: executing: app
app started up
# INFO: executing: workbook
workbook started up at http://localhost:3023/testing/
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
relative to the startup script itself.

For example, if you have a basic project in

- /tmp/foodemo

then your startup script would be at either, **but not both**, of 

- /tmp/foodemo/startup
- /tmp/foodemo/.startup/startup

and the supporting component startup scripts would be in

- /tmp/foodemo/.startup/app
- /tmp/foodemo/.startup/datafeed
- /tmp/foodemo/.startup/workbook

and you would have an optional

- /tmp/foodemo/.startup/startup.order

Apart from these files and the startup.order file described below, no other
*executable* files are allowed in the .startup directory. You can side-load
non-executable files in the .startup directory.

## Script Ordering

By default, component scripts will execute in lexicographic order. If you want to impose
a strict ordering, then create a file named startup.order in the .startup directory.

If the .startup.order file exists, and there are executable files in the .startup
directory that are not listed in it, then the startup script will provide an error
and exit when it is run.  Likewise, if you have a startup.order file, and it
refers to file that do not exist, then an error will be thrown.

In any case, the order in the startup.order file will be honored, even if the user
selects a subset fo components in a strange order.

### Optional Scripts

You may include scripts which can be run directly, but which will not be automatically
included for 'all'. In order to do this, simply comment them out in the startup.order
file like this:

~~~
script1
#optional-script2
~~~

The optional scripts will be sanity checked along with the other scripts, but they
will be excluded when 'all' scripts are run.


## Managed Scripts

If you want to include a subdirectory called `.startup/managed-scripts/`, then any
executable scripts in this directory will be available for execution. These will
not be included in `./startup all`, unless they are directly included in the
startup.order file. In this case, the scripts will be run as named in startup.order,
and if a named script is found in both places then then one in `.startup/` will take
precedence.
If you tell startup to run specific managed scripts in combination with other
scripts which *are* in the startup order, then these scripts are included only
*after* the known order.

Using these conventions, you may link in a remote repo
or directory which includes a set of managed scripts that provide default implementations. 
If there is a named script in `.startup/` which has the same name as one in
`.startup/managed-scripts/`, then the on in `.startup/` takes precedence.

## Ad-Hoc Scripts

Additional scripts may be kept below in other subdirectories besides managed-scripts, but
the relative path of these commands must be used instead of a simple script name.
Ad-hoc scripts may not be overridden, since the relative path means that they have
their own unique namespace under `.startup/some-path-...`

## Script Execution

The current working directory will be set to the parent directory of the .startup component
scripts directory before it is executed. For example, if you have $PROJECT_DIRECTORY/.startup/scriptfoo,
then before scriptfoo is run, the current working directory will be set to $PROJECT_DIRECTORY.

For integration purposes, the following environment variables will be set:

- PROJECT_DIRECTORY - The project directory that holds the .startup subdirectory. 
- SELECTED_COMPONENTS - A space-separated list of selected components in the order they were started up.
- CURRENT_COMPONENT - The component name that is currently being started.
- COMPONENT_SCRIPT - The absolute path of the script that is running the component.

## Component Script Conventions

These script conventions are fairly normative. For the purposes of uniform and reliable
startup behavior, you should consider these as strict requirements for component scripts.

- Each component script must be set to executable.
- Each non-script file must **not** be set executable.
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

