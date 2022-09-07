Grafonnet API Example
=====================

[Grafonnet](https://grafana.github.io/grafonnet-lib/) is a [Jsonnet](https://jsonnet.org/) library for programmatically generating [Grafana](https://grafana.com/) dashboards. It enables templating and easy maintenance of dashboards via source control. Generated Dashboards can be imported manually into Grafana or pushed via API.

In this example, we will cover the basic usage of Jsonnet and [Jsonnet-bundler](https://github.com/jsonnet-bundler/jsonnet-bundler), as well as the usege of Grafana's API to create and update dashboards.

For more detailed information, please refer to the [Official Documentation](https://grafana.github.io/grafonnet-lib/).

## Requirements

You will need the following:

  - Jsonnet
  - Jsonnet-bundler
  - curl

The installation depends on your operating system, please refer to the official websites for more instructions.

## Setup

Initialize the project structure by calling

```
jb init
```

This will use Jsonnet-bundler to create a `jsonnetfile.json`. This file will be used to manage Jsonnet library dependencies.

Next, we will declare our `grafonnet` dependency.

```
jb install https://github.com/grafana/grafonnet-lib/grafonnet
```

This will pull grafonnet and all its transitive dependencies into the vendor folder, ready to be imported into our jsonnet files.

Now we are ready to create our first dashboard!

## Creating a Dashboard

Here is an example empty dashboard:

```
local grafana = import 'grafonnet/grafana.libsonnet';

grafana.dashboard.new(
  title='Empty Dashboard'
)
```

You can place this in a file called `dashboard.jsonnet`. As you can see, we are importing the `grafana.libsonnet` library
with all the DSL required to programatically generate dashboards.

Now, let's convert the dashboard to JSON:

```
jsonnet -J vendor -o dashboard.json dashboard.jsonnet
```

The file `dashboard.json` has been generated. This contents of the json can be imported into Grafana manually via UI (Dashboards -> New -> Import). However, we can also do that programatically via the [Grafana Dashboards API].

## Create the Dashboard via API

Grafana provides powerful API to automate all kinds of tasks. Let's use the Dashboard API to create our Dashboard.

First of all, we need an auth token. We can create one via the Grafana UI (Configuration -> API Keys -> Add API Key).

In order to generate our request body, since we will be posting JSON, why not use Jsonnet to embed our newly generate dashboard? We can turn our previous dashboard template into a library by renaming it to `dashboard.libsonnet` and create a new jsonnet file embedding it (`api.jsonnet`) with the following contents:

```
local dashboard = import 'dashboard.libsonnet';

{
  dashboard: dashboard,
  "message": "Example dashboard",
  "overwrite": true
}
```

Once we have it, we can now generate our API request body and post our dashboard using curl:

```
jsonnet -J vendor -o api.json api.jsonnet
curl -H 'Accept: application/json' -H 'Content-Type: application/json' -X POST -H "Authorization: Bearer <our token>" -d @api.json https://<grafana host>/api/dashboards/db
```

