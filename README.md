## Second Attempt

Setup:

```
+------+      +------------+      +------------+      +------+
|client|----->|client-proxy|----->|server-proxy|----->|server|
+------+      +------------+      +------------+      +------+
```

### Verifying the setup

1. Run `docker-compose up`
2. Run `time docker exec -it envoy-respect_expected_rq_timeout-issue_client_1 curl client-proxy:9001/test`


#### Observations:
1. The request should time out in 15s
2. server-proxy logs show an inbound request with the header: `'x-envoy-expected-rq-timeout-ms', '120000'`
3. server-proxy logs show an outbound request with the header: `'x-envoy-expected-rq-timeout-ms', '15000'`, which ignores the 120000 from the egress Envoy.

Additionally, adding a route timeout to the ingress Envoy will update the timeout of the request to the server, also disregarding the value in `x-envoy-expected-rq-timeout-ms`.

Tested on versions going back to v1.14.1

### Second test
1. Run `docker-compose up`
2. Run `time docker exec -it envoy-respect_expected_rq_timeout-issue_client_1 curl -vH 'x-envoy-upstream-rq-timeout-ms: 10' client-proxy:9001/test`

#### Observations:
1. The request should time out in < 1s
2. client-proxy logs show an outbound request with the header: `'x-envoy-expected-rq-timeout-ms', '10'`
3. server-proxy logs show an inbound request with the header: `'x-envoy-expected-rq-timeout-ms', '10'`
4. server-proxy logs show an outbound request with the header: `'x-envoy-expected-rq-timeout-ms', '15000'`, which ignores the 120000 from the egress Envoy.

The behavior of the client proxy is what's expected. The upstream rq header comes in, and the client-proxy uses that as the timeout for the request to the server-proxy.
