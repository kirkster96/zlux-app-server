This program and the accompanying materials are
made available under the terms of the Eclipse Public License v2.0 which accompanies
this distribution, and is available at https://www.eclipse.org/legal/epl-v20.html

SPDX-License-Identifier: EPL-2.0

Copyright Contributors to the Zowe Project.



# zlux-app-server
This is the default setup of the Zowe App Server, built upon the zLUX framework. Within, you will find a collection of build, deploy, and run scripts as well as configuration files that will help you to configure a simple App Server and add a few Apps.

**To request features or report bugs, please use the issues page at the [zlux repo](https://github.com/zowe/zlux/issues) with the server infrastructure or server security tags**

## Server layout
At the core of the zLUX App infrastructure backend is an extensible server, written for nodeJS and utilizing expressJS for routing. It handles the backend components of Apps, and also can serve as a proxy for requests from Apps to additional servers as needed. One such proxy destination is the ZSS - a Zowe backend component called **Zowe System Services**. It's a so-called Agent for the App Server.

### ZSS & zLUX Server overlap
The zLUX App Server and ZSS utilize the same deployment & App/Plugin structure, and share some configuration parameters as well. It is possible to run ZSS and zLUX App Server from the same system, in which case you would be running under z/OS USS. This configuration requires that IBM's version of nodeJS is installed prior.

Another way to set up zLUX is to have the zLUX App Server running under LUW, while keeping ZSS under USS. This is the configuration scenario presented below. In this scenario, you'll need to clone these github repositories to two different systems, and they'll need to have compatible configurations. For first-timers, it is fine to have identical configuration files and /plugins folders in order to get going.

## First-time Installation & Use
Getting started with this server requires just a few steps:

0. [Prerequisites](#0-prerequisites)
1. [Acquire the Source Code](#1-acquire-the-source-code)
2. [Initial Build](#2-initial-build)
3. [Initial Startup](#3-initial-startup)
    1. [Server Logs](#server-logs)
4. [Connect in a Browser!](#4-connect-in-a-browser)
5. [Customizing Configuration](#5-customizing-configuration)
6. [Adding Plugins](#6-adding-plugins)
    1. [Plugin Build & Add Example](#plugin-build--add-example)
    2. [Adding More Plugins](#adding-more-plugins)
7. [Adding ZSS to the Environment](#7-adding-zss-to-the-environment)
    1. [Building ZSS](#building-zss)
    2. [Configuring App Server to Use ZSS](#configuring-app-server-to-use-zss)
    3. [Starting App Server When Using ZSS](#starting-app-server-when-using-zss)


Follow each step and you'll be on your way to your first App Server instance!

## 0. Prerequisites

### Building & Developing
To build the App Server and Apps, the following is required:

* **NodeJS** - v12.x minimum. The App Server and server plugins must support minimum v8.16.x at runtime, but v12.x is used for building.

* **npm** - v6.4 minimum

* **jdk** - v8 minimum
 
* **ant** - v1.10 minimum

* **ant-contrib** - v1 minimum

For building zss ([Section 7](#7-adding-zss-to-the-environment)):

* **IBM z/OS XLC compiler for Metal C Compilation**

For development:

* **git** - 2.18 or higher is recommended off z/os

* **ssh agent** - Our repositories are structured to expect that you have ssh keys setup for github. This assists with rapid development and automation. 
Git bash or putty's pageant are some of various tools that can help you setup & work with ssh keys over git.

#### (Optional) Install git for z/OS
Because all of our code is on github, yet ZSS must run on z/OS and the zLUX App Server may optionally run on z/OS as well, having git on z/OS is the most convenient way to work with the source code. The alternative would be to utilize FTP or another method to transfer contents to z/OS.
If you'd like to go this route, you can find git for z/OS free of charge here: 
http://www.rocketsoftware.com/product-categories/mainframe/git-for-zos

On z/OS, git 2.14.4 is the minimum needed.

### Runtime
To use the App Server, the following is required:

* **NodeJS** - v8.16.x minimum up to v12.x is officially supported by the Zowe community.

Plugins may depend upon other technologies, such as Java or ZSS. An plugin's [pluginDefinition](https://github.com/zowe/zlux/wiki/Zlux-Plugin-Definition-&-Structure) or README will help you to understand if additional prerequisites are needed for that plugin.


## 1. Acquire the source code
Afterwards, clone (or download) the github capstone repository, https://github.com/zowe/zlux
As we'll be configuring ZSS on z/OS's USS, and the zLUX App Server on a LUW host, you'll need to put the contents on both systems.
If using git, the following commands should be used:
```
git clone --recursive git@github.com:zowe/zlux.git
cd zlux
git submodule foreach "git checkout master"
```

For the initial setup, the default authentication is the "trivial authentication" plugin, which allows login to the App Server without valid credentials. At the end of this guide, you can customize the environment to switch to a secure authentication plugin instead, such as the ZSS authentication plugin, covered in [Section 7](#7-adding-zss-to-the-environment).


## 2. Initial build
Now you'll have the latest code for the server.
The App Server framework & core plugins contain server and/or web components.
Since these web components use webpack for packaging optimization, and server components may use typescript or other software that requires compiling, it is necessary to build the code before using the server and plugins.

There are utilities within the zlux-build folder for initializing & building core plugins and the framework itself, which will be used.
However, those tools are not needed when building individual plugins, as it is faster to build them each as needed when [following this documentation.](https://github.com/zowe/zlux/wiki/Building-Plugins)

On the host running the App Server, run the script that will automatically build all included Apps:

```
cd zlux-build

//Windows
build.bat

//Otherwise
./build.sh
```

This will take some time to complete.

**Note when building, NPM is used. The version of NPM needed for the build to succeed should be at least 6.4. You can update NPM by executing `npm install -g npm`**

**Note:** It has been reported that building can hang on Windows if you have put the code in a directory that has a symbolic link. Build time can depend on hardware speed, but should take minutes not hours.


Upon completion, the App Server is ready to be run.

## 3. Initial startup
To start the App Server with all default settings, do the following:
```
cd ../zlux-app-server/bin

// Windows:
app-server.bat

// Others:
./app-server.sh
```

When the App Server has started, one of the messages you will see as bootstrapping completes is that the server is listening on the HTTP/s port. Now, the server is ready for use.

### Server Logs
When the server starts, it writes logs to a text file. On z/OS, Unix, and Linux, the server also logs to the terminal via stdout.
To view the entire logs, you can find the log file within `<ZWE_zowe_logDirectory>`, which will default to `~/.zowe/logs` or `%USERPROFILE%/.zowe/logs` (Windows) if ZWE_zowe_logDirectory is not specified. The log file starts with "appServer" and the filename may also include a timestamp.

## 4. Connect in a browser
With the App Server started, you can access Apps and the Zowe Desktop from it in a web browser.
In this example, the address you will want to go to first is the location of the window management App - Zowe Desktop.
The URL for this is:

https://\<App Server\>:7556/ZLUX/plugins/org.zowe.zlux.bootstrap/web/index.html

Once here, you should be greeted with a Login screen. By default trivial authentication is used which allows to login with arbitrary credentials.
So, you can type in any username to get access to the desktop, which likely does not yet have any Apps. Next, the server should be configured, Apps added, and authentication set up.

## 5. Customizing configuration
Read the [Configuration](https://github.com/zowe/zlux/wiki/Configuration-for-zLUX-App-Server-&-ZSS) wiki page for a detailed explanation of the primary items that you'll want to configure for your server.

In short, determine the location of your workspace directory (Environment variable `ZWE_zowe_workspaceDirectory`, or `~/.zowe/workspace` (Linux, Unix, z/OS) or `%USERPROFILE%/.zowe` when `ZWE_zowe_workspaceDirectory` is not defined)
Within the workspace directory, create **app-server/serverConfig/zowe.yaml** by copying **zlux-app-server/defaults/serverConfig/zowe.yaml**, and edit it to change attributes such as the HTTPS port via **components.app-server.node.https.port**, or the location of plugins.

## 6. Adding Plugins
**Note when building, NPM is used. The version of NPM needed for the build to succeed should be at least 6.4. You can update NPM by executing `npm install -g npm`**

Plugins, like the framework, can contain server and/or web components, either of which may need to be built using technologies such as webpack and typescript.
The Plugins which have web components are also called "Apps" within the UI.
To add a plugin to the server, it must first be built. Plugins that you download may already be pre-built.

[Click here to read about building plugins](https://github.com/zowe/zlux/wiki/Building-Plugins)

[Click here to read about installing plugins](https://github.com/zowe/zlux/wiki/Installing-Plugins)

### Plugin Build & Add Example
Following the references above, this is how you can add a TN3270 terminal emulator app to your App Server, for an App server installed in `~/my-zowe`

1. `git clone https://github.com/zowe/tn3270-ng2.git ~/my-zowe/tn3270-ng2`
1. `cd ~/my-zowe/tn3270-ng2`
1. Set the environment variable `MVD_DESKTOP_DIR` to the path of zlux-app-manager/virtual-desktop, such as `export MVD_DESKTOP_DIR=~/my-zowe/zlux-app-manager/virtual-desktop`
1. `cd webClient`
1. `npm install`
1. `npm run build` (This will take some time to complete.)
1. `cd ~/my-zowe/zlux-app-server/bin`
1. `./install-app.sh ~/my-zowe/tn3270-ng2`

**Note:** If on Windows, try `%USERPROFILE%/` instead of `~/`, and `install-app.bat` instead of `./install-app.sh`

**Note:** In a production environment you should not have 3rd party apps in the same root folder as the App Server, but currently some apps have relative path limitations during build & development that lead to developing apps within the same root folder.

**Note:** It has been reported that building can hang on Windows if you have put the code in a directory that has a symbolic link. Build time can depend on hardware speed, but should take minutes not hours.

### Adding more plugins
Several plugins are available for use in Zowe. Some of them may require ZSS, which can be set up in the next steps
Here are a few plugins which you can clone to experiment with development:

- [sample-angular-app](https://github.com/zowe/sample-angular-app): A simple app showing how a zLUX App frontend (here, Angular) component can communicate with an App backend (REST) component.
- [sample-react-app](https://github.com/zowe/sample-react-app): Similar to the Angular App, but using React instead to show how you have the flexibility to use a framework of your choice.
- [sample-iframe-app](https://github.com/zowe/sample-iframe-app): Similar in functionality to the Angular & React Apps, but presented via inclusion of an iframe, to show that even pre-existing pages can be included

You can clone them all via:

```
git clone git@github.com:zowe/sample-angular-app.git
git clone git@github.com:zowe/sample-react-app.git
git clone git@github.com:zowe/sample-iframe-app.git
```

**Sample React and Iframe Apps depend on the Sample Angular App, as an example of dependencies. Recent versions of the Angular App also depends upon ZSS as an example, but you can use a version older than v1.21.0 if you do not have ZSS available.**

## 7. Adding ZSS to the environment
Like the App Server, ZSS is an HTTP(S) server component of Zowe. 
However unlike the App Server, ZSS is a z/OS specific component which exists to provide core low-level & security APIs, as well as a place to attach lower-level plugins than could be built with the App Server. The configuration, directories, and network-level API structures are similar, as these servers work together to support Apps.
Also, some of the low-level APIs are made possibly by ZSS working in concert with the Zowe Cross Memory Server, which is not an HTTP(S) server, but ZSS provides any needed HTTP(S) access. So, if you need the APIs provided by ZSS, or want to build & use low-level plugins, then you must add ZSS and the Cross Memory Server to your Zowe environment.

Since ZSS adheres to the same configuration & directory structure as the App server, the easiest way to set up ZSS is to clone all of the App Server repos ([Step 1](#1-acquire-the-source-code)) onto z/OS if you have not already done so. However, strictly speaking the only non-ZSS repo needed is the `zlux-app-server` repo because it contains the scripts that are used to start the servers.

### Building ZSS
To build ZSS from source, it is recommended to use git on z/OS. You can use this command to get the code:

```
git clone --recursive git@github.com:zowe/zss.git
cd zss/build
```

**The code must be placed on z/OS**, as ZSS can only be compiled there and will only run there.

Now, you can build both ZSS and the Cross Memory Server via:

```
./build_zss.sh
./build_zis.sh
```

A successful build will result in the ZSS binary being placed at `zss/bin/zssServer`. To use it, you need to copy `zssServer` to the `zlux-app-server/bin` directory, so that you can either run it directly via `zssServer.sh` or have the App Server automatically start it when the App Server is run via `app-server.sh`.

Before running, you must set also set program control attribute on the ZSS binary. It is needed to make the APIs run under the authenticated user's permissions. 

```
cp zssServer ../../zlux-app-server/bin
extattr +p ../../zlux-app-server/bin/zssServer
```

Finally, the ZSS Cross memory server must be installed and configured according to [This Install Guide](https://github.com/zowe/docs-site/blob/master/docs/user-guide/install-zos.md#manually-installing-the-zowe-cross-memory-server)

With ZSS built, you can now run it in one of two ways.
* Run it independently of the App Server via `zlux-app-server/bin/zssServer.sh` or
* Run the App Server and it will attempt to start `zssServer.sh` as a child process, due to it being defined as a child process in the zowe.yaml

### Configuring App Server to use ZSS
In App Server terminology, ZSS is a **Agent**, where the Agent is responsible for fulfilling low-level & OS-specific APIs that the App Server delegates. In order to use the App Server and ZSS together, your App Server must be configured to use it as an Agent, and setup with a security plugin which uses ZSS as an App Server security provider.

#### Security Provider Setup
To add ZSS as a security provider, add the **sso-auth** plugin to the App Server. Following [Section 6](#6-adding-plugins) about adding plugins, you can do the following, where `ZWE_zowe_workspaceDirectory=~/.zowe/workspace` and the App Server in `~/my-zowe`:
1. `./install-app.sh ~/my-zowe/zlux-server-framework/plugins/sso-auth`

Then, you need set the configuration of the App Server to prefer that security provider.
Locate and edit zowe.yaml (Within ZWE_zowe_workspaceDirectory/app-server/serverConfig/zowe.yaml, such as `~/.zowe/workspace/app-server/serverConfig/zowe.yaml`)
Within that file, set `dataserviceAuthentication.defaultAuthentication = "saf"`.

Keep this file open to continue with agent setup.
 
#### Agent Setup (App Server side)
Within the zowe.yaml file, you need to define or set **components.zss.agent.agent.host** to a hostname or ip address where ZSS is located that is also visible to the App Server. This could be '0.0.0.0' or the hostname of a z/OS system.
You must also define or set **components.zss.agent.agent.port**. This is the TCP port which ZSS will listen on to be contacted by the App Server. Define this in the configuration file as a value between 1024-65535. See [zss configuration](https://github.com/zowe/zlux/wiki/Configuration-for-ZLUX-App-Server-&-ZSS#zss-configuration) for more information and an example.

As a result of the above edits to zowe.yaml, an example of what it may now look like is:

```yaml
components:
  app-server:
    node:
      https:
        ipAddresses:
        - 0.0.0.0
        port: 7556
        keys:
        - "../defaults/serverConfig/zlux.keystore.key"
        certificates:
        - "../defaults/serverConfig/zlux.keystore.cer"
        certificateAuthorities:
        - "../defaults/serverConfig/apiml-localca.cer"
      childProcesses:
      - path: "../bin/zssServer.sh"
        once: true
    productDir: "../defaults"
    siteDir: "~/.zowe/workspace/app-server/site"
    groupsDir: "~/.zowe/workspace/app-server/groups"
    usersDir: "~/.zowe/workspace/app-server/users"
    pluginsDir: "~/.zowe/workspace/app-server/plugins"
    dataserviceAuthentication:
      defaultAuthentication: saf
      rbac: false
  zss:
    agent:
      host: localhost
      https:
        port: 7557

```

#### Agent Setup (ZSS side)
On z/OS, ZSS must be set to have the correct port, IP, and HTTP(S) configuration so that the app-server can reach it.

On a release install (not covered here, but described on docs.zowe.org), ZSS is already set up for use over HTTPS. You can update `ZWES_SERVER_PORT` in a Zowe zowe.yaml file to set which port it should use, to match the value you have on your dev install for `agent.https.port`.

On a dev install of ZSS, instead of zowe.yaml, server.json is used just like the dev install of the app-server. In fact if App Server and ZSS are on the same system, then this can be the same file. Otherwise, you must edit the server.json file where ZSS is and keep it in sync with the App Server one with regards to the **agent** settings. In a dev install, it is recommended to use a GSKIT compatible keyring or p12 file for using ZSS over HTTPS, or HTTPS via ATTLS, but HTTP is also possible. In that case, you simply configure `agent.http.port` instead of `agent.https.port`, and `agent.http.ipAddresses` instead of `agent.https.ipAddresses`. So, use server.json to set the port and IPs you need to make ZSS visible to the system where app-server runs.

**Note: It is highly recommended to have HTTPS for ZSS. This is the default on a release install, but it is also possible to use AT-TLS to do this such as by following [configuring AT-TLS](https://zowe.github.io/docs-site/latest/user-guide/mvd-configuration.html#configuring-zss-for-https). It is most important when using ZSS externally, as the session security is essential for all but trivial development environments**

#### Agent swagger installation
If you intend to use the API Catalog as well, you will want to [install the plugin](https://github.com/zowe/zlux/wiki/Installing-Plugins) "zlux-agent", found in `zlux-server-framework/plugins/zlux-agent`.
This allows the app-server to serve agent swagger definitions, which are then visible in the API catalog.


### Starting App Server when using ZSS
When running the App Server with an Agent such as ZSS, you can either set the `server.json` with all info to connect to the Agent, or set that info via environment variable or command line arguments. The above sections detail `server.json` configuration, but the other ways to set the IP and port of the Agent are:

1. Environment variable configuration
Items in server.json can be substitued by environment variables that have a name that corresponds to each item. The pattern is 

`ZWED_json_key=value` where `ZWED_` is the prefix, and `json_key` is the name of the key. For example,

```
ZWED_agent_http_port=9999
ZWED_node_https_ipAddresses=127.0.0.1,192.168.1.100
``` 

This would override the `server.json` value of `agent.http.port` to be 9999, and `node.https.ipAddresses` to be ['127.0.0.1','192.168.1.100'].

2. CLI argument configuration
Items in server.json can also be substituded by arguments and flags. The pattern for arguments is

`-Djson.key=value`, where `-D` denotes an argument, and `json.key` is the name of the key. For example,

`-Dagent.http.port=9999 -Dnode.https.ipAddresses=127.0.0.1,192.168.1.100` Would set `agent.http.port` to be 9999, and `node.https.ipAddresses` to be ['127.0.0.1,'192.168.1.100'].

There are also specific flags used for well-known configuration items, such as:
- *-h*: Declares the host where ZSS can be found. Use as "-h \<hostname\>"
- *-P*: Declares the port at which ZSS is listening.
- *-s*: Declares the https port the App Server will listen on.



This program and the accompanying materials are
made available under the terms of the Eclipse Public License v2.0 which accompanies
this distribution, and is available at https://www.eclipse.org/legal/epl-v20.html

SPDX-License-Identifier: EPL-2.0

Copyright Contributors to the Zowe Project.
