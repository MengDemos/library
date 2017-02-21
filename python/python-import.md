You can define a global name __all__, set to a list or tuple of names, to limit what is imported and what documentation tools generally will list:

__all__ = ['function1', 'ClassName2']
The __all__ name limits what from test import * will import, and is also used by documentation tools to limit what is listed as public API for a given module.

See the import statement documentation:

The public names defined by a module are determined by checking the module’s namespace for a variable named __all__; if defined, it must be a sequence of strings which are names defined or imported by that module. The names given in __all__ are all considered public and are required to exist. If __all__ is not defined, the set of public names includes all names found in the module’s namespace which do not begin with an underscore character ('_'). __all__ should contain the entire public API. It is intended to avoid accidentally exporting items that are not part of the API (such as library modules which were imported and used within the module).
The __init__ modules you inspected will almost certainly define __all__ sequences.

You can also delete names again from your module, provided your functions do not need access to the global names later:

del sys
The IPython autocompletion otherwise uses all names that do not start with an underscore; the autocompletion ignores the __all__ list, but will ignore names like _sys.

The numpy.__init__ module (before version 1.8.0) itself deletes names from the global namespace again:

if __NUMPY_SETUP__:
    import sys as _sys
    _sys.stderr.write('Running from numpy source directory.\n')
    del _sys
but here sys is bound as _sys and IPython would ignore that name even if it wasn't deleted. numpy also builds up an __all__ list in that module.

In numpy version 1.8.0 and newer, a import sys statement was added to that file and IPython offers it for autocompletion, because it is still part of the global namespace.
