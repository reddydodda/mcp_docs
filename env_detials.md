# IP with Port numbers :
-------------------------
## Salt
Salt IP : 10.100.0.15
Salt api port : 6969


## Jenkins
Jenkins on Salt Node : 10.100.0.15:8081

Jenkins on CICD Node : 10.101.0.90:8081

## Gerrit
Git path on Salt node : /root/github

Gerrit on CICD node : 10.101.0.90:8080

## OpenStack

Dashboard URL : 10.101.0.80 ( proxy node vip )
Admin endpoint : 10.101.0.10

## Stacklight

Kibana : 10.101.0.80:5601

Prometheus UI : 10.101.0.70:15010  ( obtain vip by salt 'mon01*'  pillar.get _param:keepalived_prometheus_vip_address)
