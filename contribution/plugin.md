How plugins work
----------------

All plugins in the `libraries/plugins` directory are iterated over by `CMakeLists.txt` and placed in a CMake environment variable `STEEMIT_INTERNAL_PLUGINS`, which is used to create a runtime-accessible list of available plugins used by the argument parsing.

Similarly, `external_plugins` is set aside for third-party plugins.  Just drop plugin code into `external_plugins` directory, `make steemd`, and the new plugin will be available.

There is a plugin in `example_plugins` called `hello_api` which is a working example of adding a custom API call.

Registering plugins
-------------------

- Plugins are enabled with the `enable-plugin` config file option.
- When specifying plugins, you should specify `witness` and `account_history` in addition to the new plugins.
- Some plugins may keep records in the database (currently only `account_history` does).  If you change whether such a plugin is disabled/enabled, you should also replay the chain.  Detecting this situation and automatically replaying when needed will be implemented in a future release.
- If you want to make API's available publicly, you must use the `public-api` option.
- When specifying public API's, you should specify `database_api` and `login_api` in addition to the new plugins.
- The `api-user` option allows for password protected access to an API.