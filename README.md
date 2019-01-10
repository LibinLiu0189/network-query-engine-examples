
# Forward Data Model Export

Forward Networks Data Model Export (DME) provides a simple API that exposes information about the network as JSON data in a
fully-parsed form. The information is normalized and presented uniformly across devices from different vendors. The
exported data structures are standards-aligned with [OpenConfig](http://www.openconfig.net/) (details below), and
all data queries are available through a [GraphQL](https://graphql.org/) API. This API allows network operators to
easily develop scripts - for example, to perform sanity checks or to display information - that work across the entire
fleet of devices in their network.

This repository provides examples to help you get started using Data Model Export.

# Getting Started

## Try Queries on the Demo Network

The quickest way to get started is to try out sample queries against a demo network on [Forward App](https://fwd.app/). Note that if
you do not already have an account there, you can
[request an account here](https://www.forwardnetworks.com/network-mapping-software/).

Browse to https://fwd.app/data-model-explorer. Use network and snapshot selectors in the toolbar to select the demo
network and its snapshot. Now try out the following simple query, by dropping this into the left hand-side pane and
running the play button:
```
{
  devices {
    name
  }
}
```

You should see output like this in the right hand-side pane, containing all the device names in the demo network:
```
{
  "data": {
    "devices": [
      {
        "name": "atl-app-lb01"
      },
      {
        "name": "atl-ce01"
      },
      ...
    ]
  }
}
```

You can try out other queries. The editor provides assistance via auto-complete and inline help, which you can access
by pressing control+shift keys. In addition, you can explore the full schema, including detailed formal and informal
documentation, by clicking on the "Docs" link in the top-right-hand side toolbar.

Note that you can try out these queries against your own network snapshot by choosing the appropriate network and
snapshot on the query window.

## Queries via curl

You can also query DME using curl (or any other tool that can issue HTTP requests). The API is available by HTTP POST to
the following URL
```
https://<ForwardInstance>/api/snapshots/<SnapshotId>/graphql
```
The POST should include a GraphQL query as a JSON object in the following form:
```
{ "query": "<your query>", "variables": [{"var1", "value1"}, ..., {"varn", "valuen"}]}
```
You can omit the `variables` field if you do not need to pass any variables. See
https://graphql.org/learn/queries/#variables for background info on variables.

For example, you can issue the query shown previously via curl against https://fwd.app as follows:

```
curl --user myusername:mypassword https://fwd.app/api/snapshots/100/graphql -X POST -H "Content-Type: application/json" -d '{"query": "{ devices { name } }"}'
```

## Queries via Python

This repository includes a simple client library that can be used to run queries. Install the library like this:
```
sudo pip install .
```

Now you can run the examples in the examples/ directory. For example, try navigating to the examples directory, and then
run:
```
python mismatched_interfaces.py https://fwd.app <yourUsername> <yourPassword> <yourSnapshotId>
```

# Examples

The repository consists of a few examples
* Get device names.
* Find interfaces that are configured up, but are not up.
* Find IP addresses that are assigned to more than one interface within a VRF.

# Relationship to OpenConfig

Forward Networks DME is *aligned* with OpenConfig, but is not literally the same data model as any of the various
JSON representations of the OpenConfig YANG data models. Rather, Forward Networks DME can be seen as an idiomatic
representation of OpenConfig as a GraphQL data source: we have adapted the OpenConfig YANG data models to fit within
the constraints of GraphQL, to adapt to the conventions prevalent within GraphQL APIs, and to take advantage of the
powereful query features available in GraphQL.

Specifically, Forward Networks DME differs from OpenConfig models in the following ways:
1. Names are camel-cased. Dashes are not permitted in GraphQL.
2. The config and state hierarchy is squashed out, similar to "path-compression" in
[ygot](https://github.com/openconfig/ygot/blob/master/docs/design.md#openconfig-path-compression).
3. OpenConfig sometimes represents some complex entity as a string with specific format, such as vlan range in the
`x..y` format. DME chooses to model these as structured objects.
4. Some collections of elements, such as BGP routes, are "paged" in DME because they are often very large. This paging
allows clients to ask for a page of routes at a time.
5. Additional data is sometimes added, such as device platform information under the `platform` field of `Device`, or
the `links` field of an interface that adds topology information to the interface object information.
6. DME takes advantage of GraphQL's *graph* model to provide fields on objects that link (aka *join*) to related
information. For example, a field of an interface links to other interface to which it is connected. This field
provides the related interface object, not just its name.

Forward Networks DME only covers a subset of OpenConfig models. Specifically, it covers the following:
1. Interface data
2. Network instance data
3. BGP data (presently limited `adj-rib-in` and `adj-rib-out`)

# Contributing

We welcome contributions! Please submit and share scripts, queries, and examples. Bug fixes, doc improvements and so on
are also welcome. To contribute, fork the repository, push your work to a branch in that repository and then submit a
pull request. Specifically, do:

1. Fork this repo
2. Clone your forked repo
2. In your forked repo, create your feature branch (`git checkout -b my-feature`)
3. Push to the new branch of your forked repo: (`git push origin my-feature`)
4. Create new Pull Request

## Contact

[@AndreasVoellmy](@AndreasVoellmy) or use the project GitHub issues.

# Further reading

* [Product docs](https://app.forwardnetworks.com/docs/applications/data_model_export/)
* [Data Model Export Blog](https://app.forwardnetworks.com/TBD)
