## Loki Access

[Loki](https://github.com/grafana/loki#loki-like-prometheus-but-for-logs) is a horizontally-scalable, highly-available, multi-tenant log aggregation system.

An instance of Loki is available for use in OperateFirst.

Loki access is open to public and is divided into 3 different components:

1. Loki Distributor
2. Loki Frontend
3. Grafana

While pushing logs into Loki or while querying Loki for logs,
you will need to have a header `'X-Scope-OrgID'` set in your requests. <br>
This header (OrgID) is used to identify tenants in a multi tenancy setup ([more info](https://grafana.com/docs/loki/latest/operations/multi-tenancy/)),
it will also play a role when adding a Loki Grafana datasource, more on that later.

The value for this header can be anything. <br>
For our examples, we will set it to `'X-Scope-OrgID: opf-example'`.

### Loki Distributor

- url: http://loki-distributor-opf-observatorium.apps.zero.massopen.cloud/

This is the endpoint that can be used to push logs into Loki. <br>

Example:

```bash
curl -v -H "Content-Type: application/json" \
  -H "X-Scope-OrgID: opf-example" \
  -XPOST -s "http://loki-distributor-opf-observatorium.apps.zero.massopen.cloud/loki/api/v1/push" --data-raw \
  '{"streams": [{ "stream": { "app": "my-app-1" }, "values": [ [ "1607526347000000000", "my-log-line" ] ] }]}'
```

This command sends a log line `"my-log-line"` with the label `"app": "my-app-1"` for the timestamp `1607526347000000000` (`2020-12-09T15:05:47.000Z`) and uses `"opf-example"` as the OrgID (or tenant ID).

More Info on [/push](https://grafana.com/docs/loki/latest/api/#post-lokiapiv1push)

### Loki Frontend

- url: http://loki-frontend-opf-observatorium.apps.zero.massopen.cloud/

This is the endpoint that can be used to query logs from Loki. <br>

Example:

```bash
curl -H "X-Scope-OrgID: opf-example" \
  -G -s  "http://loki-frontend-opf-observatorium.apps.zero.massopen.cloud/loki/api/v1/query_range" \
  --data-urlencode 'query={app="my-app-1"}' \
  --data-urlencode 'start=1607526347000000000'
```

This command queries logs from Loki using `"app="my-app-1"` label as the
query and the start time for the query is set to `1607526347000000000` (`2020-12-09T15:05:47.000Z`). <br>
_Notice that we had to provide the same OrgID "opf-example" to be able to query for the logs that we pushed in._

More Info on [/query_range](https://grafana.com/docs/loki/latest/api/#get-lokiapiv1query_range)

### Grafana

- url: https://grafana-opf-observatorium.apps.zero.massopen.cloud/

This is the endpoint for Grafana that can be used to query logs from Loki in a UI.

To be able to query logs using Grafana, you will need to add a Grafana Datasource
specific to your Org-ID (the `'X-Scope-OrgID'` header). To do that instructions are available [here](./add_loki_grafana_datasource.md).

An example query should be accessible at the following url: <br>
https://grafana-opf-observatorium.apps.zero.massopen.cloud/explore?orgId=1&left=["1607519717000","1607526347000","loki-opf-example",{"expr":"{app%3D\"my-app-1\"}"}]
