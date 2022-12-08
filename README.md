$ vi docker-compose.tls.yml

$ vi create-certs.yml

Generate certificates for Elasticsearch by bringing up the create-certs container.
$ docker-compose -f create-certs.yml run --rm create_certs
Enter password for CA Private key :1234
Enter password for CA (/certs/elastic-stack-ca.p12) : 1234
ls ./certs
elastic-certificates.p12  elastic-stack-ca.p12

$ vi .env.es

$ vi .env.apm

Run the Elasticsearch cluster in docker
$ docker-compose -f docker-compose.tls.yml --profile es up -d
