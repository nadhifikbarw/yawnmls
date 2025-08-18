# Elasticsearch

Single-node Elastic cluster + Kibana, with HTTPS enabled
to provide development environment closer to production cluster.

- Hide away certificates configuration between Kibana and Elastic
- Provide ready-to-use `elastic` user with password you want

This compose configuration is explicit, and convenient for tighter development environment
if you already use `compose.yaml` for other service dependencies.

I prefer having this `compose.yaml` compared to following instruction from
https://www.elastic.co/docs/deploy-manage/deploy/self-managed/install-elasticsearch-docker-basic
where it requires manual steps to connect Kibana.

Or https://github.com/elastic/start-local/blob/main/start-local.sh where it requires extra
step to setup bootstrap your compose file.

- `http://localhost:5601` to access Kibana (no HTTPS). Use `elastic` and `es-password`
- `https://localhost:9200` to connect to ElasticSearch instance (Use HTTPS)

# CA cert

To communicate securely over HTTPS. You can use `docker cp` to copy
`ca.crt` from `/usr/share/elasticsearch/config/certs/ca/ca.crt`, or even simpler
using VSCode extension to explore container with such volume, download it once
and put it in `./dev/certs/ca` for your application code

You then use this certificate to verify Elasticsearch server certificate
for your application client.
