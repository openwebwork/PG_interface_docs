# PG Rendering API

This is the basic documentation for a renderer wrapper around the pg translate function. 
The goal is to create a module/class that can be used throughout webwork as well as more standalone services that render PG problems. 

The hope is also to pass into the function a minimal number of flags/parameters/arguments that is needed for rendering as well as 
the flexibility for the function to handle any of the current situations, especially the current `webwork2` platform as well as other 
services that are currently being created. 


```
function render {
  my ($problem, $input, $envir, $flags) = @_; 


}
```

The inputs for the function are

* __problem__: a problem from either the OPL, a local directory or as a string.  It should have the following fields
  - `global`: boolean whether or not the problem is global (from OPL)
  - `path`: string of the path (either local or global)
  - `seed`: integer of the problem seed
  - `source`: string as a source of a problem. 
  - `course_dir`(if a problem is local)

* __input__: a hash of the input to the problem.  Basically the current `formFields` variable.

* __envir__: a hash of any needed environmental flags.  See [Environment variables wiki page](https://github.com/openwebwork/PG_interface_docs/wiki/PG-Renderer-Environmental-Variables) page for more information.

* __flags__: a hash of additional flags needed for the problem. See the [Processing Flags](https://github.com/openwebwork/PG_interface_docs/wiki/PG-renderer-processing-flags) page for more information.


The output of the function is a hash very similar to the current hash:

* `head_text`
* `post_header_text`
* `body_text`
* `answers`
* `result`
* `state`
* `errors`
* `warnings`
* `flags`
* `pgcore`
