<img src="http://zenoh.io/img/zenoh-dragon-small.png" width="150">

[![CI](https://github.com/eclipse-zenoh/zenoh-plugin-webserver/workflows/CI/badge.svg)](https://github.com/eclipse-zenoh/zenoh-plugin-webserver/actions?query=workflow%3A%22CI%22)
[![Gitter](https://badges.gitter.im/atolab/zenoh.svg)](https://gitter.im/atolab/zenoh?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge)
[![License](https://img.shields.io/badge/License-EPL%202.0-blue)](https://choosealicense.com/licenses/epl-2.0/)
[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)

# Web Server plugin for Eclipse zenoh

The Web Server plugin implements an HTTP server mapping URLs to zenoh paths.  
This plugin can be used to set-up a Web server where the resources are retrieved from geo-distributed
zenoh storages, each leveraging various backends (file system, database, memory...).

**Library name** `zplugin_webserver`

**Startup arguments** (added to zenoh router’s startup arguments):
 - `--web-server-port=[PORT]`: The Web Server plugin's http port (default: 80)

:point_right: **Download:** https://download.eclipse.org/zenoh/zenoh-plugin-webserver/

-------------------------------
## **Examples of usage**

Assuming you have a static website stored in `/var/www/html` directory, you can:
 - expose the files as zenoh key/values using the [File System backend](https://github.com/eclipse-zenoh/zenoh-backend-filesystem)
 - set-up this Web Server plugin that will allow HTTP clients to browse the files.

Here are the steps:
 1. Make sure the libraries for the File System backend and the Web Server plugin are available for the zenoh router:  
    either installing their packages (depending your platform), either downloading the library files corresponding
    to your platform in your `~/.zenoh/lib` directory.
 2. Start the zenoh router (`zenohd`). It will automatically load the Web Server plugin and make it available on port 80.
 3. Add the File System backend (for instance using `curl` on the REST API):
    ```bash
    curl -X PUT http://localhost:8000/@/router/local/plugin/storages/backend/fs
    ```
 4. Add a File System storage exposing the `/var/www/html` directory in read-only mode under the `/my-site` zenoh prefix:
    ```bash
    curl -X PUT -H 'content-type:application/properties' -d "path_expr=/my-site/**;path_prefix=/my-site;dir=/var/www/html;read_only" http://localhost:8000/@/router/local/plugin/storages/backend/fs/storage/my-site
    ```
 5. Now you can browse your site on http://localhost/my-site.


For more advanced use cases you can also:
 - Have the files of your web sites stored on different hosts. Running a zenoh router with a File System Storage on
   each host allow to make all the files available under the `/my-site` zenoh path.
 - Duplicate the files of your web sites on several hosts to provide fault tolerance.
 - Start several zenoh routers with the Web Service plugin on different hosts (not necessarly the same than the
   hosts running the File System storages). Each host will serve your web site.
 - Use other backends than the File System to store your resources and make them available through zenoh.
   (List of available backends [here](http://zenoh.io/docs/manual/backends-list/)).
 - Deploy a zenoh application that will implement `eval` function for a resource, replying to requests with a
   dynamic content (see the `z_eval` example in
   [Rust](https://github.com/eclipse-zenoh/zenoh/blob/master/zenoh/examples/zenoh/z_eval.rs) or
   [Python](https://github.com/eclipse-zenoh/zenoh-python/blob/master/examples/zenoh/z_eval.py)).



-------------------------------
## How to build it

At first, install [Cargo and Rust](https://doc.rust-lang.org/cargo/getting-started/installation.html). 

:warning: **WARNING** :warning: : As Rust doesn't have a stable ABI, the backend library should be
built with the exact same Rust version than `zenohd`. Otherwise, incompatibilities in memory mapping
of shared types between `zenohd` and the library can lead to a `"SIGSEV"` crash.

To know the Rust version you're `zenohd` has been built with, use the `--version` option.  
Example:
```bash
$ zenohd --version
The zenoh router v0.5.0-beta.5-134-g81e85d7 built with rustc 1.51.0-nightly (2987785df 2020-12-28)
```
Here, `zenohd` has been built with the Rust toolchain version **nightly-2020-12-28**.  
Install and use this toolchain with the following command:

```bash
$ rustup default nightly-2020-12-28
```

And then build the backend with:

```bash
$ cargo build --release --all-targets
```
