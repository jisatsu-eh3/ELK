# Tutorial: Securing a Self-Managed Elastic Stack with a Custom Local CA 

## Prerequisites
- Custom CA running on a separate Windows VM.
- Each ELK component (Elasticsearch, Kibana, Logstash) is on its own Red Hat VM.

## Step 1: Generate Certificates on the CA VM (Windows)

## Generate Certificates for Each ELK Component

For Elasticsearch:

```
openssl genpkey -algorithm RSA -out elasticsearch.key -pkeyopt rsa_keygen_bits:2048
openssl req -new -key elasticsearch.key -out elasticsearch.csr -subj "/C=US/ST=State/L=City/O=Organization/OU=OrgUnit/CN=elasticsearch.example.com"
openssl x509 -req -in elasticsearch.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out elasticsearch.crt -days 365 -sha256
```

For Kibana:

```
openssl genpkey -algorithm RSA -out kibana.key -pkeyopt rsa_keygen_bits:2048
openssl req -new -key kibana.key -out kibana.csr -subj "/C=US/ST=State/L=City/O=Organization/OU=OrgUnit/CN=kibana.example.com"
openssl x509 -req -in kibana.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out kibana.crt -days 365 -sha256
```

For Logstash:

```
openssl genpkey -algorithm RSA -out logstash.key -pkeyopt rsa_keygen_bits:2048
openssl req -new -key logstash.key -out logstash.csr -subj "/C=US/ST=State/L=City/O=Organization/OU=OrgUnit/CN=logstash.example.com"
openssl x509 -req -in logstash.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out logstash.crt -days 365 -sha256
```

### Expected Output:

    For each component:
        <component-name>.key (Private key)
        <component-name>.csr (Certificate Signing Request)
        <component-name>.crt (Signed certificate)

## Step 2: Distribute Certificates to ELK VMs
### Copy CA Certificate to Each ELK VM

Use a secure copy tool to copy the CA certificate to each ELK component VM.

Example using PowerShell with OpenSSH:

```
scp ca.crt user@elasticsearch-vm:/etc/elasticsearch/certs/
scp ca.crt user@kibana-vm:/etc/kibana/certs/
scp ca.crt user@logstash-vm:/etc/logstash/certs/
```

### Copy Component Certificates to Respective VMs

Use a secure copy tool to transfer the component certificates and keys.

Example using PowerShell with OpenSSH:

```
scp elasticsearch.crt elasticsearch.key user@elasticsearch-vm:/etc/elasticsearch/certs/
scp kibana.crt kibana.key user@kibana-vm:/etc/kibana/certs/
scp logstash.crt logstash.key user@logstash-vm:/etc/logstash/certs/
```


### Update Elasticsearch Configuration (elasticsearch.yml)

```
xpack.security.transport.ssl.enabled: true
xpack.security.transport.ssl.verification_mode: certificate
xpack.security.transport.ssl.key: /etc/elasticsearch/elasticsearch.key
xpack.security.transport.ssl.certificate: /etc/elasticsearch/elasticsearch.crt
xpack.security.transport.ssl.certificate_authorities: [ "/etc/elasticsearch/ca.crt" ]

xpack.security.http.ssl.enabled: true
xpack.security.http.ssl.key: /etc/elasticsearch/elasticsearch.key
xpack.security.http.ssl.certificate: /etc/elasticsearch/elasticsearch.crt
xpack.security.http.ssl.certificate_authorities: [ "/etc/elasticsearch/ca.crt" ]
```

### Update Kibana Configuration (kibana.yml)

```
server.ssl.enabled: true
server.ssl.certificate: /etc/kibana/kibana.crt
server.ssl.key: /etc/kibana/kibana.key
elasticsearch.ssl.certificateAuthorities: [ "/etc/kibana/ca.crt" ]
```


### Update Logstash Configuration (logstash.yml and pipeline configuration)

```
# logstash.yml
xpack.monitoring.elasticsearch.ssl.certificate_authority: "/etc/logstash/ca.crt"

# pipeline configuration (example)
output {
  elasticsearch {
    hosts => ["https://elasticsearch.example.com:9200"]
    ssl => true
    cacert => "/etc/logstash/ca.crt"
    user => "elastic"
    password => "changeme"
  }
}
```













