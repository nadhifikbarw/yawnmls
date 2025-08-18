# Elasticsearch

Single-node Elasticsearch cluster + Kibana, with HTTPS enabled
to provide development environment closer to production cluster.

- Hide away certificates configuration between Kibana and Elastic
- Provide ready-to-use `elastic` user with password you provide
- `http://localhost:5601` to access Kibana (no HTTPS). Use `elastic` user
- `https://localhost:9200` to connect to ElasticSearch instance (Use HTTPS)

This `compose.yaml` provides explicit and more convenient tighter development
environment, if you already use `compose.yaml` for other service dependencies
this can support your workflow further.

I prefer having centralized `compose.yaml` to setup local development environment
instead of having to manually setup https://www.elastic.co/docs/deploy-manage/deploy/self-managed/install-elasticsearch-docker-basic where it requires manual steps
to connect Kibana to Elasticsearch.

Or even using https://github.com/elastic/start-local/blob/main/start-local.sh,
where it's sufficient when you need quick Elasticsearch cluster but
it provides fragmented experience for application development.

# CA cert

To communicate securely over HTTPS. You can use `docker cp` to copy
`ca.crt` from `/usr/share/elasticsearch/config/certs/ca/ca.crt` from
`elasticsearch` service.

What I do that's simpler is just use Docker VSCode extension to download
`ca.crt` file once and put it in `./dev/certs/ca` for your application code,
or wherever you need it.

![Download ca.crt using VSCode extension](/assets/es-ca-cert.png)

Even simpler workflow is disable certificate verification for your application
elasticsearch client in development.
