# Elasticsearch

Single-node Elastic cluster + Kibana, with HTTPS enabled
to provide development environment closer to production cluster.

This compose configuration is convenient for tighter developer environment
if you use other service dependencies already configured in your `compose.yaml`

I prefer this `compose.yaml` compared to instruction from
https://www.elastic.co/docs/deploy-manage/deploy/self-managed/install-elasticsearch-docker-basic
where it requires manual steps to connect Kibana.

- `http://localhost:5601` to access Kibana (no HTTPS). Use `elastic` and `es-password`
- `https://localhost:9200` to connect to ElasticSearch instance (Use HTTPS)

# CA cert

To communicate securely over HTTPS. You can use `docker cp` to copy
`ca.crt` from `/usr/share/elasticsearch/config/certs/ca/ca.crt`, or simpler
use VSCode extension to explore container with such volume, download it once
and put it in `./dev/certs/ca`

You then use this certificate to verify Elasticsearch server certificate
for your application client.
