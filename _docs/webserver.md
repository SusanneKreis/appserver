---
layout: docs
title: Webserver
meta_title: appserver.io webserver
meta_description: appserver.io comes with a integrated webserver. This is build and configured like any other server component using our multithreaded server framework.
position: 40
group: Docs
subNav:
  - title: Connection Handler
    href: connection-handler
  - title: Server Modules
    href: server-modules
  - title: Virtual Hosts
    href: virtual-hosts
  - title: Environment Variables
    href: environment-variables
  - title: Authentications
    href: authentications
  - title: Accesses
    href: accesses
  - title: File Handlers
    href: file-handlers
  - title: Locations
    href: locations
  - title: Rewrites
    href: rewrites
  - title: VirtualHost Examples
    href: virtualhost-examples
permalink: /get-started/documentation/webserver.html
---

The Webserver is build and configured like any other server component using our
[multithreaded server framework](<https://github.com/appserver-io/server>). Let's have a look at the main configuration
of the server component itselfe.

```xml
<server
  name="http"
  type="\AppserverIo\Server\Servers\MultiThreadedServer"
  worker="\AppserverIo\Server\Workers\ThreadWorker"
  socket="\AppserverIo\Server\Sockets\StreamSocket"
  serverContext="\AppserverIo\Server\Contexts\ServerContext"
  requestContext="\AppserverIo\Server\Contexts\RequestContext"
  loggerName="System">
```

There are several attributes to configure for a server component which are described in the following
table.

| Attributes        | Description |
| ----------------- | ----------- |
| `name`            | The name of the server component used for reference and logging purpose. |
| `type`            | The server type implementation classname based on `AppserverIo\Server\Interfaces\ServerInterface`. It provides the main daemon like logic of the server. |
| `worker`          | The worker queue implementation classname based on `\AppserverIo\Server\Interfaces\WorkerInterface`. It will introduce a common worker queue logic for the server being able processing many requests at the same time. This could be either a classic eventloop or a threaded, forked mechanism |
| `socket`          | The socket implementation classname based on `AppserverIo\Psr\Socket\SocketInterface`. It provides common socket functionality. As we have our [psr for sockets](<https://github.com/appserver-io-psr/socket>) now you might have a look at it. |
| `serverContext`   | The server context implementation classname based on `\AppserverIo\Server\Interfaces\ServerContextInterface`. It represents the server context while running as daemon and holds the configuration, loggers and an optional injectable container object which can be used to connect several server components. |
| `requestContext`  | The request context implementation classname based on `\AppserverIo\Server\Interfaces\RequestContextInterface`. It holds all vars needed (server, environment and module vars) which can be processed and modified by the server-module-chain defined. After the request was pre processed by internal server-modules the request context will provide those information for specific file-handlers being able to process the request in a common way. |
| `loggerName`      | The logger instance to use in the server's context. |

Next thing we'll have a look at are server params.

```xml
<params>
    <param name="admin" type="string">info@appserver.io</param>
    <param name="software" type="string">appserver/1.0.0-beta4.19 (linux) PHP/5.5.19</param>
    <param name="transport" type="string">tcp</param>
    <param name="address" type="string">127.0.0.1</param>
    <param name="port" type="integer">9080</param>
    <param name="workerNumber" type="integer">64</param>
    <param name="workerAcceptMin" type="integer">3</param>
    <param name="workerAcceptMax" type="integer">8</param>
    <param name="documentRoot" type="string">webapps</param>
    <param name="directoryIndex" type="string">index.do index.php index.html index.htm</param>
    <param name="keepAliveMax" type="integer">64</param>
    <param name="keepAliveTimeout" type="integer">5</param>
    <param name="errorsPageTemplatePath" type="string">var/www/errors/error.phtml</param>
</params>
```

They are used to define several key/value pairs for the Webserver implementation to react on. Find the param
descriptions below.

| Param                    | Description |
| ------------------------ | ----------- |
| `admin`                  | The email address of the administrator who is responsible for. |
| `software`               | The software signature as showen in the response header for example. |
| `transport`              | The transport layer. In ssl mode `ssl` will be used instead of plain `tcp`. |
| `address`                | The address the server-socket should be bind and listen to. If you want to allow only connection on local loopback define ´127.0.0.1´ as in the example above shown. This will be good enough for local development and testing purpos. If you want to allow connections to your external ethernet interfaces just define `0.0.0.0` or if you want to allow connection only on a specific interface just define the ip of your interface `192.168.1.100`. |
| `port`                   | The port for the server-socket to accept connections to. Default setting is `9080` and `9443` for ssl. If you want to serve through default web ports just define `80` and for https `443`. Make sure there is no other Webserver installed blocking the default ports.|
| `workerNumber`           | Defines the number of worker-queues to be started waiting for requests to process. |
| `workerAcceptMin`        | Describes the minimum number of requests for the worker to be accepted for randomize its lifetime. |
| `workerAcceptMax`        | Describes the maximum number of requests for the worker to be accepted for randomize its lifetime. |
| `documentRoot`           | Defines the root directory for the server to append the uri with and search for the requested file or directory. The document root path will be relative to the servers root directory if there is no beginning slash "/" |
| `directoryIndex`         | Defines the index resources to lookup for the requested directory. The server will return the first one that it finds. If none of the resources exist, the server will respond with a 404 Not Found. |
| `keepAliveMax`           | The number of requests allowed per connection when keep-alive is on. If it is set to 0 keep-alive feature will be deactivated. |
| `keepAliveTimeout`       | The number of seconds waiting for a subsequent request while in keep-alive loop before closing the connection. |
| `errorsPageTemplatePath` | The path to the errors page template. The path will be relative to the servers root directory if there is no beginning slash "/". |

If you want to setup a Webserver you have to configure 2 more params.

| Param         | Description |
| ------------- | ----------- |
| `certPath`    | The path to your certificate file which as to be a combined PEM file of private key and certificate. The path will be relative to the servers root directory if there is no beginning slash "/". |
| `passphrase`  | The passphrase you have created your SSL private key file with. Can be optional. |

## Connection Handler

As we wanted to handle requests based on a specific protocol, the server needs a mechanism to understand and handle
those requests in a proper way.

For our Webserver we use `\AppserverIo\WebServer\ConnectionHandlers\HttpConnectionHandler`
which implements the `\AppserverIo\Server\Interfaces\ConnectionHandlerInterface` and follows the HTTP/1.1 specification,
which can be found [here](<http://tools.ietf.org/html/rfc7230>) using our [HTTP library](<https://github.com/appserver-io/http>).

```xml
<connectionHandlers>
    <connectionHandler type="\AppserverIo\WebServer\ConnectionHandlers\HttpConnectionHandler" />
</connectionHandlers>
```

> There is the possibility to provide more than one connection handler, but in most cases it does not make sense because
> you will have to handle another protocol which might not be compatible with the modules you provided in the same server
> configuration. In certain circumstances it will make sense but it's not best practise to do this.

## Server Modules

As mentioned at the beginning we're using our [multithreaded server framework](<https://github.com/appserver-io/server>)
which allows you to provide modules for request and response processing triggered from several hooks.

Let's get an overview of those hooks which can also be found in the corresponding dictionary class `\AppserverIo\Server\Dictionaries\ModuleHooks`

| Hook             | Description |
| ---------------- | ----------- |
| `REQUEST_PRE`    | The request pre hook should be used to do something before the request will be parsed. So if there is a keep-alive loop going on this will be triggered every request loop. |
| `REQUEST_POST`   | The request post hook should be used to do something after the request has been parsed. Most modules such as CoreModule will use this hook to do their job. |
| `RESPONSE_PRE`   | The response pre hook will be triggered at the point before the response will be prepared for sending it to the to the connection endpoint. |
| `RESPONSE_POST`  | The response post hook is the last hook triggered within a keep-alive loop and will execute the modules logic when the response is well prepared and ready to dispatch. |
| `SHUTDOWN`       | The shutdown hook is called whenever a php fatal error will shutdown the current worker process. In this case current filehandler module will be called to process the shutdown hook. This enables the module the possibility to react on fatal error's by it's own in some cases. If it does not react on this shutdown hook, the default error handling response dispatcher logic will be used. If the module reacts on the shutdown hook and set's the response state to be dispatched no other error handling shutdown logic will be called to fill up the response. |

Now let's dig into the modules list provided for the Webserver by default.

```xml
<modules>
    <!-- REQUEST_POST hook -->
    <module type="\AppserverIo\WebServer\Modules\VirtualHostModule"/>
    <module type="\AppserverIo\WebServer\Modules\AuthenticationModule"/>
    <module type="\AppserverIo\WebServer\Modules\EnvironmentVariableModule" />
    <module type="\AppserverIo\WebServer\Modules\RewriteModule"/>
    <module type="\AppserverIo\WebServer\Modules\DirectoryModule"/>
    <module type="\AppserverIo\WebServer\Modules\AccessModule"/>
    <module type="\AppserverIo\WebServer\Modules\CoreModule"/>
    <module type="\AppserverIo\WebServer\Modules\PhpModule"/>
    <module type="\AppserverIo\WebServer\Modules\FastCgiModule"/>
    <module type="\AppserverIo\Appserver\ServletEngine\ServletEngine" />
    <!-- RESPONSE_PRE hook -->
    <module type="\AppserverIo\WebServer\Modules\DeflateModule"/>
    <!-- RESPONSE_POST hook -->
    <module type="\AppserverIo\Appserver\Core\Modules\ProfileModule"/>
</modules>
```

For every hook all modules are processed in the same order as they are listed in the xml configuration.
> The order of the modules provided by the default configuration is intended and should not be changed. For example if you change the
> order of AccessModule to come before the RewriteModule it would be possible to lever an access rule by any rewrite rule.

Our Webserver provides an interface called `\AppserverIo\WebServer\Interfaces\HttpModuleInterface` that every module has
to implement.

Find an overview of all modules below ...

| Module                      | Description |
| --------------------------- | ----------- |
| `VirtualHostModule`         | Provides virtual host functionality that allows you to run more than one hostname (such as yourname.example.com and othername.example.com) on the same server while having different params and configurations. |
| `AuthenticationModule`      | Offers the possibility to secure resources using basic or digest authentication based on request uri with regular expression support. |
| `EnvironmentVariableModule` | This module let you manipulate server environment variables. These can be conditionally set, unset and copied in form of an OS context. |
| `RewriteModule`             | A simple rewrite module for PHP based web servers which uses a self made structure for usable rules. It can be used similar to Apaches mod_rewrite and provides rewriting and redirecting capabilities. |
| `DirectoryModule`           | Provides for "trailing slash" redirects and serving directory index files. |
| `AccessModule`              | Allows a HTTP header based access management with regular expression support. |
| `CoreModule`                | HTTP server features that are always available such as serving static resources and finding defined file handlers. |
| `PhpModule`                 | Acts like a classic php Webserver module (such as `mod_php` for apache) which calls and runs your requested php scripts in an isolated context with all globals (such as `$_SERVER`, `$_GET`, `$_POST` etc.) prepared in the common way. |
| `FastCgiModule`             | The Module allows you to connect several fastcgi backends (such as `php-fpm` or `hhvm`) based on configured file-handlers. |
| `ServletEngine`             | The ServletEngine introduces a super fast and simple way to implement an entry point to handle HTTP requests that allows you to execute all performance critical tasks. Please see [Servlet Engine](<{{ "/get-started/documentation/servlet-engine.html" | prepend: site.baseurl }}>) for full documentation. |
| `DeflateModule`             | It provides the `deflate` output filter that allows output from your server to be compressed before being sent to the client over the network. |
| `ProfileModule`             | Allows request based realtime profiling using external tools like logstash and kibana. |

## Virtual Hosts

Using virtual hosts you can extend the default server configuration and produce a host specific
environment for your hostname or app to run.

You can do so by adding a virtual host configuration to your global server configuration file. See
the example for a XML based configuration below:

```xml
<virtualHosts>
  <virtualHost name="example.local">
    <params>
      <param name="admin" type="string">
        admin@appserver.io
      </param>
      <param name="documentRoot" type="string">
        /opt/appserver/webapps/example
      </param>
    </params>
  </virtualHost>
</virtualHosts>
```

The above configuration sits within the server element and opens up the virtual host `example.local`
which has a different document root than the global configuration has. The virtual host is born. :-)

> Most of the `params` that are available in the `server` node can be overwritten and you can also define all following
> configurations like [Environment Variables](<#configure-your-environment-variables>), [Authentications](<#configure-authentications>), [Accesses](<#configure-accesses>) and of course [Locations](<#configure-locations>) for every virtual host
> on its own.

The `virtualHost` element can hold params, rewrite rules or environment variables which are only 
available for the host specifically.

## Environment Variables

You can set environment variables using either the global or the virtual host based configuration.
The example below shows a basic usage of environment variables in XML format.

```xml
<environmentVariables>
  <environmentVariable 
    condition="" 
    definition="EXAMPLE_VAR=example" />
  <environmentVariable 
    condition="Apple@$HTTP_USER_AGENT" 
    definition="USER_HAS_APPLE=true" />
</environmentVariables>
```

There are several ways in which this feature is used. You can get a rough idea when having a 
look at Apache modules [mod_env](<http://httpd.apache.org/docs/2.2/mod/mod_env.html>) and 
[mod_setenvif](<http://httpd.apache.org/docs/2.2/mod/mod_setenvif.html>) which we adopted.

You can make definitions of environment variables dependent on REGEX based conditions which will
be performed on so called backreferences. These backreferences are request related server variables
like `HTTP_USER_AGENT`.

A condition has the format `<REGEX_CONDITION>@$<BACKREFERENCE>`. If the condition is empty the 
environment variable will be set every time.

The definition you can use has the form `<NAME_OF_VAR>=<THE_VALUE_TO_SET>`. The definition has 
some specialities too:

- Setting a var to `null` will unset the variable if it existed before
- You can use backreferences for the value you want to set as well. But those are limited to 
  environment variables of the PHP process
- Values will be treated as strings

## Authentications

You can setup request uri based basic or digest authentication with regular expression support using authentications.
Here is an example how to configure basic or digest auth.

```xml
<authentications>
    <authentication uri="^\/auth\/basic\/.*">
        <params>
            <param name="type" type="string">
                \AppserverIo\WebServer\Authentication\BasicAuthentication
            </param>
            <param name="realm" type="string">
                PhpWebServer Basic Authentication System
            </param>
            <param name="file" type="string">
                var/www/auth/basic/.htpasswd
            </param>
        </params>
    </authentication>
    <authentication uri="^\/auth\/digest\/.*">
        <params>
            <param name="type" type="string">
                \AppserverIo\WebServer\Authentication\DigestAuthentication
            </param>
            <param name="realm" type="string">
                appserver.io Digest Authentication System
            </param>
            <param name="file" type="string">
                var/www/auth/digest/.htpasswd
            </param>
        </params>
    </authentication>
</authentications>
```

As you can see every `authentication` node has it `uri` attribute where you can use regular expressions for a request
uri to match. It also has some params which are descripted below.

| Module  | Description |
| ------- | ----------- |
| `type`  | The type represents an implementation of the `\AppserverIo\WebServer\Interfaces\AuthenticationInterface` which provides specific auth mechanism. You can use the predelivered classes `\AppserverIo\WebServer\Authentication\BasicAuthentication` and `\AppserverIo\WebServer\Authentication\BasicAuthentication` for basic and digest authentication. |
| `realm` | The string assigned by the server to identify the protection space of the request uri. |
| `file`  | The path to your credentials .htpasswd file. The path will be relative to the servers root directory if there is no beginning slash "/". |

## Accesses

You can easily allow or deny access on resources based on client's HTTP request headers by setting up accesses within
your server or virtual-host configuration.

```xml
    <accesses>
        <!-- per default allow everything -->
        <access type="allow">
            <params>
                <param name="X_REQUEST_URI" type="string">.*</param>
            </params>
        </access>
    </accesses>
```

All `access` nodes need to have a type which can be either `allow` or `deny`. To react on specific request headers and
their values you have to add a one `param` node per header. The `name` attribute has to be the request header name and
the value is a regular expression you want to match.

Everything is denied if there are no accesses configured. That's the reason why you'll find an allow all access rule
in the appserver.xml by default. That means that you can either allow everything and deny specific things or just allow
specific things. Deny rules will overload access rules!

> All request header value checks (means all `param` nodes), given by an `access` node are AND combined.
> The default behaviour is to deny everything if there are no accesses configured.

## File Handlers

File handlers are used for define a mapping between specific [Server Modules](<#server-modules>) and requested
resources by their file extensions.

```xml
<fileHandlers>
    <fileHandler name="fastcgi" extension=".php">
        <params>
            <param name="host" type="string">127.0.0.1</param>
            <param name="port" type="integer">9010</param>
        </params>
    </fileHandler>
<fileHandlers>
```

If you use this configuration a client requesting a resource having the extension `.php` will be processed by the
fastcgi server module. That means instead of serving the `.php` file as static resource by the core module the fastcgi
module will proceed the request by connecting to a fastcgi backend provided in the corresponding `params` node.

| Param  | Description |
| ------ | ----------- |
| `host` | The ip address to the fastcgi backend |
| `port` | The port to the fastcgi backend |

> The file handlers name have to be equal to the modules name you want to trigger. So every module has to implement
> a `getModuleName()` method as defined in `\AppserverIo\Server\Interfaces\ModuleInterface`.

## Locations

Locations are usefull if you want to have other fileHandlers or the fileHandlers configuration changed on a
certain request uri pattern to be triggered.

```xml
<locations>
    <location condition="\/test\.php">
        <handlers>
            <handler name="fastcgi" extension=".php">
                <!--
                <params>
                    <param name="host" type="string">127.0.0.1</param>
                    <param name="port" type="integer">9555</param>
                </params>
                 -->
            </handler>
        </handlers>
    </location>
</locations>
```

In this example the `/test.php` script will be processed by another fastcgi backend listening on `127.0.0.1:9555`

## Rewrites

Of course you can do rewriting as well! Just have a look at this simple but well known rewrite example where all
requests will be moved to an `index.php` script.

```xml
<rewrites>
    <rewrite condition="-f" target="" flag="L" />
    <rewrite condition="^/(.*)$" target="index.php" flag="L" />
</rewrites>
```

All rewrites are based on rewrite rules which consist of three important parts:

- *condition string* : Conditions which have to be met in order for the rule to take effect. See more [down here](<#condition-syntax>)

- *target string* : The target to rewrite the requested URI to. Within this string you can use backreferences similar
    to the Apache mod_rewrite module with the difference that you have to use the `$ syntax`
    (instead of the `$/%/%{} syntax` of Apache).
    Backreferences are parts of the matching rule conditions which you specifically pick out via regex.

    *Simple example* : A condition like `(.+)@$X_REQUEST_URI` would produce a back reference `$1` with the value `/index`
        for a requested URI `/index`. The target string `$1/welcome.html` would therefore result in a rewrite to `/index/welcome.html`

- *flag string* : You can use flags similar to mod_rewrite which are used to make rules react in a certain way or
    influence further processing. See more [down here](<#flags>)

### Condition Syntax

The Syntax of possible conditions is roughly based on the possibilities of Apache's RewriteCondition and RewriteRule
combined.
To make use of such a combination you can chain conditions together using the `{OR}` symbol for OR-combined, and the `{AND}`
symbol for AND-combined conditions.
Please be aware that AND takes precedence over OR!
Conditions can either be PCRE regex or certain fixed expressions.
So a condition string of `([A-Z]+\.txt){OR}^/([0-9]+){AND}-f` would match only real files (through `-f`) which either begin
with numbers or end with capital letters and the extension .txt.
As you might have noticed: Backslashes do **not have to be escaped**.

You might also be curious of the `-f` condition.
This is a direct copy of Apaches -f RewriteCondition.
We also support several other expressions to regex based conditions which are:

 - *<<COMPARE_STRING>* : Is the operand lexically preceding `<COMPARE_STRING>`?
 - *><COMPARE_STRING>* : Is the operand lexically following `<COMPARE_STRING>`?
 - *=<COMPARE_STRING>* : Is the operand lexically equal to `<COMPARE_STRING>`?
 - *-d* : Is the operand a directory?
 - *-f* : Is the operand a real file?
 - *-s* : Is the operand a real file of a size greater than 0?
 - *-l* : Is the operand a symbolic link?
 - *-x* : Is the operand an executable file?

If you are wondering what the `operand` might be: it is **whatever you want it to be**!
You can specify any operand you like using the `@` symbol.
All conditions within a rule will use the next operand to their right and if none is given the requested URI.
For example:

- *`([A-Z]+\.txt){OR}^/([0-9]+)`* Will take the requested URI for both conditions (note the `{OR}` symbol)
- *`([A-Z]+\.txt){OR}^/([0-9]+)@$DOCUMENT_ROOT`* Will test both conditions against the document root
- *`([A-Z]+\.txt)@$DOCUMENT_ROOT{OR}^/([0-9]+)`* Will only test the first one against the document root and the second against the requested URI

You might have noted the `$` symbol before `DOCUMENT_ROOT` and remembered it from the backreference syntax.
That's because all Apache common server vars can be explicitly used as backreferences too!

That does not work for you? Need the exact opposite? No problem!

All conditions, weather regex or expression based can be negated using the `!` symbol in front of them!
So `!^([0-9]+)` would match all strings which do NOT begin with a number and `!-d` would match all non-directories.

### Flags

Flags are used to further influence processing.
You can specify as many flags per rewrite as you like, but be aware of their impact!
Syntax for several flags is simple: just separate them with a `,` symbol.
Flags which might accept a parameter can be assigned one by using the `=` symbol.
Currently supported flags are:

- *L* : As rules are normally processed one after the other, the `L` flag will make the flagged rule the last one processed
    if matched.

- *R* : If this flag is set we will redirect the client to the URL specified in the `target string`. If this is just an URI we will redirect to the same host.
    You might also specify a custom status code between 300 and 399 to indicate the reason for/kind of the redirect. Default is `301` aka `permanent`

- *M* : Stands for map. Using this flag you can specify an external source (have a look at the Injector classes of the Webserver project) of a target map.
    With `M=<MY_BACKREFERENCE>` you can specify what the map's index has to match. This matching is done **only** if the rewrite condition matches and will behave as another condition

## VirtualHost Examples

The following examples should help you to configure your legacy application with default settings usually
provided with the applications .htaccess files.

### Magento

```xml
<virtualHost name="magento.dev magento.local">
    <params>
        <param name="documentRoot" type="string">webapps/magento</param>
    </params>
    <rewrites>
        <rewrite condition="-d{OR}-f{OR}-l" target="" flag="L" />
        <rewrite condition="(.*)" target="index.php/$1" flag="L" />
    </rewrites>
    <accesses>
        <access type="allow">
            <params>
                <param name="X_REQUEST_URI" type="string">
                    ^\/([^\/]+\/)?(media|skin|js|index\.php).*
                </param>
            </params>
        </access>
    </accesses>
</virtualHost>
```

### TYPO3 Neos

```xml
<virtualHost name="neos.dev neos.local">
    <params>
        <param name="documentRoot" type="string">webapps/neos-1.0.2/Web</param>
    </params>
    <rewrites>
        <rewrite
            condition="^/(_Resources/Packages/|robots\.txt|favicon\.ico){OR}-d{OR}-f{OR}-l"
            target="" flag="L" />
        <rewrite
            condition="^/(_Resources/Persistent/[a-z0-9]+/(.+/)?[a-f0-9]{40})/.+(\..+)"
            target="$1$3" flag="L" />
        <rewrite condition="^/(_Resources/Persistent/.{40})/.+(\..+)"
            target="$1$2" flag="L" />
        <rewrite condition="^/_Resources/.*" target="" flag="L" />
        <rewrite condition="(.*)" target="index.php" flag="L" />
    </rewrites>
    <environmentVariables>
        <environmentVariable condition=""
            definition="FLOW_REWRITEURLS=1" />
        <environmentVariable condition=""
            definition="FLOW_CONTEXT=Production" />
        <environmentVariable condition="Basic ([a-zA-Z0-9\+/=]+)@$Authorization"
            definition="REMOTE_AUTHORIZATION=$1" />
    </environmentVariables>
</virtualHost>
```

### ORO CRM

```xml
<virtualHost name="oro-crm.dev oro-crm.local">
    <params>
        <param name="documentRoot" type="string">webapps/crm-application/web
        </param>
    </params>
    <rewrites>
        <rewrite condition="-f" target="" flag="L" />
        <rewrite condition="^/(.*)$" target="app.php" flag="L" />
    </rewrites>
</virtualHost>
```

### Wordpress

```xml
<virtualHost name="wordpress.local">
    <params>
        <param name="documentRoot" type="string">webapps/wordpress</param>
    </params>
</virtualHost>
```
