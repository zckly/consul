# Guide to Hacking the Consul Web UI

## Last edit by Zack 3/10/17

### Top level explanation 

The Consul UI is primarily responsible for a GUI view of the Datacenters, Services, Nodes, and Key/Value pairs that are currently registered in Consul. The UI receives and works with this data through Consul’s HTTP API.

### Where to start

I would begin by defining the first view, Services. This view shows all the current services recognized by Consul. To fetch this data, make a GET request to the  endpoint [/v1/catalog/services](https://www.consul.io/docs/agent/http/catalog.html#catalog_services). 

To add a service, make a PUT request to the endpoint [/v1/agent/service/register](https://www.consul.io/docs/agent/http/agent.html#agent_service_register) with a JSON object as the data. If you're getting a weird formatting error, try JSON.stringifying the object.

Next up is the Nodes view. This view shows all the current nodes recognized by Consul. To fetch this data, make a GET request to the endpoint [/v1/catalog/nodes](https://www.consul.io/docs/agent/http/catalog.html#catalog_nodes).

To add a node, simply make a PUT request to the endpoint (/v1/catalog/register)[https://www.consul.io/docs/agent/http/catalog.html#catalog_register] with a JSON object as input. Once again, if you get a formatting error, trying stringifying the JSON.

Next is the Key Value storage view. Consult the Consul HTTP API docs for K/V for information on interactions with this endpoint: [https://www.consul.io/docs/agent/http/kv.html]

### After creating the new view

After you finish creating the new UI, we need to be able to access it with consul. To start the Consul UI, we need to pass in the `-ui-dir` flag into the `consul agent` CLI command. For example:

`consul agent -ui-dir /usr/local/go/src/github.com/hashicorp/consul/ui -data-dir /tmp/ -server` 

Below this guide is Consul's guide for the Web UI. You probably don't need any of it if you build your own.


## Consul Web UI

This directory contains the Consul Web UI. Consul contains a built-in
HTTP server that serves this directory, but any common HTTP server
is capable of serving it.

It uses JavaScript and [Ember](http://emberjs.com) to communicate with
the [Consul API](https://www.consul.io/docs/agent/http.html). The basic
features it provides are:

- Service view. A list of your registered services, their
health and the nodes they run on.
- Node view. A list of your registered nodes, the services running
on each and the health of the node.
- Key/value view and update

It's aware of multiple datacenters, so you can get a quick global
overview before drilling into specific data-centers for detailed
views.

The UI uses some internal undocumented HTTP APIs to optimize
performance and usability.

### Development

Improvements and bug fixes are welcome and encouraged for the Web UI.

You'll need sass to compile CSS stylesheets. Install that with
bundler:

    cd ui/
    bundle

Reloading compilation for development:

    make watch

Consul ships with an HTTP server for the API and UI. By default, when
you run the agent, it is off. However, if you pass a `-ui-dir` flag
with a path to this directory, you'll be able to access the UI via the
Consul HTTP server address, which defaults to `localhost:8500/ui`.

An example of this command, from inside the `ui/` directory, would be:

    consul agent -bootstrap -server -data-dir /tmp/ -ui-dir .

Basic tests can be run by adding the `?test` query parameter to the
application.

When developing Consul, it's recommended that you use the included
development configuration.

    consul agent -config-file=development_config.json

### Releasing

`make dist`

The `../pkg/web_ui` folder will contain the files you should use for deployment.

###Acknowledgements
cog icon by useiconic.com from the [Noun Project](https://thenounproject.com/term/setting/45865/)

### Compiling the UI into the Go binary

The UI is compiled and shipped with the Consul go binary. The generated bindata
file lives in the `command/agent/bindata_assetfs.go` file and is checked into
source control. This is useful so that not every Consul developer needs to set
up bundler etc. To re-generate the file, first follow the compilation steps
above to build the UI assets into the `pkg/web_ui` folder. With that done, from the
root of the Consul repo, run:

```
$ make static-assets
```

The file will now be refreshed with the current UI data. You can now rebuild the
Consul binary normally with `make bin` or `make dev`, and see the updated UI
after re-launching Consul.
