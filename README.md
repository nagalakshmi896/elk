$ vi docker-compose.tls.yml

$ vi create-certs.yml

Generate certificates for Elasticsearch by bringing up the create-certs container.
$ docker-compose -f create-certs.yml run --rm create_certs
Enter password for CA Private key :1234
Enter password for CA (/certs/elastic-stack-ca.p12) : 1234
ls ./certs
elastic-certificates.p12  elastic-stack-ca.p12

$ vi .env.es
{
# Basic configs
cluster.initial_master_nodes=es01,es02
cluster.name=es-docker-cluster
bootstrap.memory_lock=true
ES_JAVA_OPTS=-Xms512m -Xmx512m

# Security Configs
xpack.security.enabled=true
xpack.security.transport.ssl.enabled=true
xpack.security.transport.ssl.keystore.type=PKCS12
xpack.security.transport.ssl.verification_mode=certificate
xpack.security.transport.ssl.keystore.path=elastic-certificates.p12
xpack.security.transport.ssl.truststore.path=elastic-certificates.p12
xpack.security.transport.ssl.truststore.type=PKCS12

xpack.security.audit.enabled=true
ELASTIC_PASSWORD=zpUd7M366c4qrChm3xce
}

$ vi .env.apm
{
output.elasticsearch.hosts=http://es01:9200
SERVER_PUBLICBASEURL=http://10.2.3.28:5601
}


$ vi .env.kibana
{
ELASTICSEARCH_HOSTS=http://es01:9200
#SERVER_NAME=kibana.example.com
ELASTICSEARCH_USERNAME=kibana_system
ELASTICSEARCH_PASSWORD=xxxxxxxxxxx
# server.publicBaseUrl
SERVER_PUBLICBASEURL=https://kibana.example.com/
# xpack.encryptedSavedObjects.encryptionKey
XPACK_ENCRYPTEDSAVEDOBJECTS_ENCRYPTIONKEY=xxxxxxxxxxx
}

sudo sysctl -w vm.max_map_count=262144

Run the Elasticsearch cluster in docker
$ docker-compose -f docker-compose.tls.yml --profile es up -d

after five minutes run below command 
!! WARNING !! At this point, Kibana cannot connect to the Elasticsearch cluster. We must generate a password for the built-in kibana_system user. 
Run elasticsearch-setup-passwords tool to generate passwords for all built-in users, including the kibana_system user.
$ docker exec es01 /bin/bash -c "bin/elasticsearch-setup-passwords auto --batch --url http://es01:9200"
output
{
Changed password for user apm_system
PASSWORD apm_system = gqrn5lLCC24eijYBGdpo

Changed password for user kibana_system
PASSWORD kibana_system = mkxGEDWIWsYeBAo3Ye5d

Changed password for user kibana
PASSWORD kibana = mkxGEDWIWsYeBAo3Ye5d

Changed password for user logstash_system
PASSWORD logstash_system = LqFBjFIkeu7YowRbXo6W

Changed password for user beats_system
PASSWORD beats_system = yS4KZNwySjK7SCxnvB58

Changed password for user remote_monitoring_user
PASSWORD remote_monitoring_user = N49elTVdby0SL72DJ7Dw

Changed password for user elastic
PASSWORD elastic = 6GFI4ufHKnRjO6QKLzdu
}

$ vi .env.kibana (enter above generated elastic user and passwords below)

{
ELASTICSEARCH_HOSTS=http://es01:9200
#SERVER_NAME=kibana.example.com
ELASTICSEARCH_USERNAME=elastic
ELASTICSEARCH_PASSWORD=6GFI4ufHKnRjO6QKLzdu
# server.publicBaseUrl
SERVER_PUBLICBASEURL=https://13.235.94.135:5601
# xpack.encryptedSavedObjects.encryptionKey
XPACK_ENCRYPTEDSAVEDOBJECTS_ENCRYPTIONKEY=DtHA&A6b"|2L8a"-PxI+S^2&!,fmKjs,40^DM=>/
}
note generete your own pasword(DtHA&A6b"|2L8a"-PxI+S^2&!,fmKjs,40^DM=>/)


$ mkdir configs
$ cd configs/
$ vi apm-server.yml
{
apm-server:
  host: "0.0.0.0:8200"
  auth:
    secret_token: 9r#f+`p15Ij$AyFj<sY|QPPoV/!m4_
  rum:
    enabled: true
  # To enable remote configuration
  kibana:
    enabled: true
    host: "https://13.235.94.135:5601"

# https://www.elastic.co/guide/en/apm/server/current/configuring-output.html
output:
  elasticsearch:
    hosts:
      - es01:9200
    username: "elastic"
    password: "6GFI4ufHKnRjO6QKLzdu"

queue.mem.events: 4096

max_procs: 4
}
note:elastic ,6GFI4ufHKnRjO6QKLzdu generated passwords in above steps

Use docker-compose to start the Kibana:
$ docker-compose -f docker-compose.tls.yml --profile es --profile kibana up -d

http://13.235.94.135:9200/  elastic search
username elastic
password 6GFI4ufHKnRjO6QKLzdu

http://13.235.94.135:5601/login?next=%2F kibana
username elastic
password 6GFI4ufHKnRjO6QKLzdu

$ docker-compose -f docker-compose.tls.yml --profile logging  up -d   (to create APM server)

















