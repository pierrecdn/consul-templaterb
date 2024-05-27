# consul-ui

A simple HTML5 app that displays all services within Consul with AJAX requests.
It supports fitering based on tags and does not rely on Consul to display pages,
meaning that it can be scaled horizontally without any troubles with Consul.

## Features

* List all services details, with very fast lookups.
* `consul-timeline-ui.html` Let you see all changes applied to your services with history
* List all keys
* List all nodes
* List datacenters

## Quick install with Docker

An Docker image is also available https://hub.docker.com/r/discoverycriteo/consul-templaterb
and allows to quickly have a working
[Consul-UI](https://github.com/criteo/consul-templaterb/blob/master/samples/consul-ui/README.md)
that will server the UI to explore your Consul Cluster.

## Is it prod ready?

This application is used for several months within Criteo to replace Consul native interface and
is used daily by hundreds of users. At Criteo, consul-templaterb creates the static web, updates
it continuously and all the static pages are served using nginx, but any web server might be used.

Criteo has more than 20k Consul nodes on 8 datacenters, so it should work for most installations.

It has far more performance than most Consul UIs, the only downside is that the whole application
is Read-Only (but it would be possible to add RW support with CORS to modify values within
Consul).

You can watch a [video comparison](https://www.youtube.com/watch?v=o7VEox2FSEs) of this application
with the new native Consul UI interface of Consul 1.2.x, performance gap is especially strong
when using remote Datacenters (in the video around 01:10) where this app displays all information
from datacenter in Tokyo from Paris.

## Usage

```shell
consul-templaterb -c http://localhost:8500 samples/consul-ui/*.erb
```

Where `http://localhost:8500` is the address of the Consul agent you want to use to
perform requests.

The content is statically created, so you can serve it using any HTTP server very easily.

For development, you might use `python -m SimpleHTTPServer` in order to server the result
on your browser.

### Running it in production

Whatever your solution, be sure to have a index.html, so read next below on
how generating an index.html.

#### Running with web server

You can run it with your favoite web server, at Criteo we run it with nginx
which handles nicely cache and offer good performance.

In that case, consul-templaterb can start nginx with `--exec` or you can run it
as a daemon, in which case, consul-templaterb only generate the files.

#### Run it in Consul

You can run consul with `-ui-dir=/path/to/directory/of/consul-ui`, in such case
reaching consul on poort 8500 will redirect to the /ui/ path, displaying the UI
of your choice on http://consul-agent.example.org:8500/ui/.

### Generating index.html

By default, the command `consul-templaterb -c http://localhost:8500 samples/consul-ui/*.erb``
will generate sample/consul-ui/consul-services-ui.html file. If you want to use index.html instead
you might use the `--template samples/consul-ui/consul-services-ui.html:samples/consul-ui/index.html`
instead. Example:

```shell
consul-templaterb -c http://localhost:8500 \
  --template samples/consul-ui/consul-services-ui.html.erb:samples/consul-ui/index.html \
  samples/consul-ui/*.erb
```

Will generate index.html and consul_template.json in your directory, so you might serve it directly.

### Filtering services

This app supports the following environment variables:

* `SERVICES_TAG_FILTER`: basic tag filter for service (default HTTP)
* `INSTANCE_MUST_TAG`: Second level of filtering (optional, default to SERVICES_TAG_FILTER)
* `INSTANCE_EXCLUDE_TAG`: Exclude instances having the given tag (default: canary)
* `EXCLUDE_SERVICES`: comma-separated services to exclude (default: lbl7.*,netsvc-probe.*,consul-probed.*)
* `CONSUL_TIMELINE_BUFFER`: number of entries to keep in the timeline. 1000 by default.
* `CONSUL_TIMELINE_BLACKLIST`: regexp of services to hide from timeline

### Preferences

Some templates allows you to override some behavior through `.preferences.rb` file.
In order to do that, rename `.preferences.rb.samples` file to `.preferences.rb`
and update lambda defined inside to match targeted behavior.
