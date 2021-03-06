# [scons2ninja: Generate Ninja build files for SCons](https://el-tramo.be/scons2ninja)

## About

This script generates a [Ninja](http://martine.github.io/ninja/) build file for a
[SCons](http://scons.org) project.

## Usage

First of all, you need to do a small change to the `SConstruct` file of your project.
This script assumes that the `SConstruct` has a configuration variable 
`dump_trace`, which, when set, executes the following:

    env.SetOption('no_exec', True)
    env.Decider(lambda x, y, z: True)
    SCons.Node.Python.Value.changed_since_last_build = (lambda x, y, z: True)

(i.e. a dry run of rebuilding all files). Also make sure the command line
is fully printed.

To generate the initial `build.ninja` file, run the following from the
toplevel directory:

    path/to/scons2ninja.py <optional scons flags>

This will generate the `build.ninja` file for an equivalent SCons build with the given flags. 
From then on, you can just type `ninja` to build.
The `build.ninja` file will be automatically regenerated when necessary (i.e. when a 
`SCons*` file changes).

The script also reads in a `.scons2ninja.conf` file in the current directory, which can
be used to hook your own custom steps into the process. See the example configuration file
to see some customizations.

## Caveats

- This script isn't fully generic yet, so there are still pretty big holes in customization and
  configuration.
- This script has only been tested with a very limited set of tools (gcc, clang, Visual Studio,
  Qt, ...). Since the script requires full
  knowledge of all tools run by SCons, it will fail if it finds an unsupported tool. Feel free
  to send a patch with support for your favorite tools.
- The script does not take into account environment variables etc. set from SCons, and
  the ninja build will use the current environment instead.
- Files that have dynamic content (i.e. content depending on other things than files, such as
  `Value`s), will not be regenerated automatically. This is a general problem with generator-based
  build systems. Delete the file if you want it to be regenerated.

## TODO

- Use the batching feature for SCons-generated files, when/if it is implemented in Ninja.
  This allows to build all SCons-generated files in one SCons invocation, saving SCons startup
  time for every file.

## Other projects

NaCl recently [added their own script](https://groups.google.com/forum/#!msg/native-client-dev/Ev06azGi7pI/2bGp2h_kY-UJ) for integrating Ninja with SCons. It plugs in directly into the SCons backend to get dependency information,
which means it does not require full knowledge of all tools used in the build process. I will look into following
a similar approach for scons2ninja, which would make it more generic and work better out of the box.
