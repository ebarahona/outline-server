# Outline Server Creator

[![Build Status](https://travis-ci.org/Jigsaw-Code/outline-server.svg?branch=master)](https://travis-ci.org/Jigsaw-Code/outline-server)

This repository has all the code needed to create and manage Outline servers on DigitalOcean. An Outline server runs
instances of Shadowsocks proxies and provides an API used by the Outline Manager application.


## Components

The system comprises the following components:

- **Server Manager Electron App:** an Election application that wraps the Server Manager web application and runs
  natively on Desktop. It adds some extra functionality, like validation of the server self-signed certificate and interception of the DigitalOcean registration flow.

  Code: `src/server_manager/electron_app`
- **Proxy Server**: a server that runs the Shadowsocks instances and a REST API to manage its users. Used as backend by the
  Server Manager app.

  Code: `src/shadowbox`

## Server Manager

### Setup

Ensure you have the following installed:
  - [Node](https://nodejs.org/)
  - [Yarn](https://yarnpkg.com/en/docs/install)
  - [Wine](https://www.winehq.org/download), if you would like to generate binaries for Windows.

Install dependencies:
```
yarn
```

If you are using root (not recommended on your dev machine, maybe in a container), you need to add `{ "allow_root": true }` to your `/root/.bowerrc` file.


### Electron App

To run the electron app:
```
yarn do server_manager/electron_app/run
```

To build the app for all platforms:
```
yarn do server_manager/electron_app/package
```

The per-platform standalone apps will be at `build/electron_app/bundled`.

The per-platform standalone apps packaged for distribution will be at
`build/electron_app/packaged` in the following formats:

- Windows: zip files. Only generated if you have [wine](https://www.winehq.org/download) installed.
- Linux: tar.gz files.
- macOS: dmg files if built from macOS, zip files otherwise.

To perform a release, use
```
yarn do server_manager/electron_app/release
```

This will perform a clean and reinstall all dependencies to make sure the build is not tainted.


## Proxy Server

See [`src/shadowbox/README.md`](src/shadowbox/README.md).

## Unit Tests

To run all the tests, run `yarn test`


## Build System

We have a very simple build system based on package.json scripts that are called using `yarn`
and a thin wrapper for what we call build "actions".

We've defined a `do` package.json script that takes an `action` parameter:
```shell
yarn do $ACTION
```

This command will define a `do_action()` function and call `${ACTION}_action.sh`, which must exist.
The called action script can use `do_action` to call its dependencies. The $ACTION parameter is
always resolved from the project root, regardless of the caller location.

The idea of `do_action` is to keep the build logic next to where the relevant code is.
It also defines two environmental variables:

- ROOT_DIR: the root directory of the project, as an absolute path.
- BUILD_DIR: where the build output should go, as an absolute path.

### Build output

Building creates the following directories under `build/`:
- `web_app/`: The Manager web app.
  - `static/`: The standalone web app static files. This is what one deploys to a web server or runs with Electron.
- `electron_app/`: The launcher desktop Electron app
  - `static/`: The Manager Electron app to run with the electron command-line
  - `bundled/`: The Electron app bundled to run standalone on each platform
  - `packaged/`: The Electron app bundles packaged as single files for distribution
- `invite_page`: the Invite Page
  - `static`: The standalone static files to be deployed
- `shadowbox`: The Proxy Server

The directories have subdirectories for intermediate output:
- `ts/`: Autogenerated Typescript files
- `js/`: The output from compiling Typescript code
- `browserified/`: The output of browserifying the Javascript code

To clean up:
```
yarn run clean
```
