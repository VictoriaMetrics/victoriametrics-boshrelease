# Victoria Metrics BOSH release 
This bosh release helps you to deploy victoria-metrics tsdb and provide monitoring solution for your bosh managed environment.
## Prerequisites:
1. bosh environment
2. bosh upload-release 
3. bosh stemcells
## Victoria-metrics deployment cases:
<details>
      <summary>Standalone victoria-metrics tsdb with basic auth and tls.</summary>
               
      bosh -d victoria-metrics deploy manifests/victoria-metrics.yml \
            -o manifests/operators/enable-tls.yml \
            -v system_domain=sys.domain \
            -v apps_domain=apps.domain \
            -v skip_ssl_verify=true \
            -o manifests/operators/singlevm.yml

* You can use [manifests/operators/gcp_ops.yaml ](https://github.com/VictoriaMetrics/victoriametrics-boshrelease/blob/master/manifests/operators/gcp_ops.yaml) as an example to adjust bosh cloud-config
      
      bosh -d victoria-metrics deploy manifests/victoria-metrics.yml \
      -o manifests/operators/gcp_ops.yaml \
      -o manifests/operators/enable-tls.yml \
      -v system_domain=sys.domain \
      -v apps_domain=apps.domain \
      -v skip_ssl_verify=true \
      -o manifests/operators/singlevm.yml
     
Depends on your network configuration you can access victoriametrics UI on https://vm-ip-address:8428/.
- Username - admin
- Password check in credhub with
```bash 
 credhub get --name /bosh/victoria-metrics/victoria_metrics_password
```
</details>

<details>
      <summary>Victoria-metrics deployment with grafana and route registration in CF.</summary>

* Create UAA client for bosh-exporter by applying [manifests/operators/bosh/add-bosh-exporter-uaa-clients.yml ](https://github.com/VictoriaMetrics/victoriametrics-boshrelease/blob/master/manifests/operators/bosh/add-bosh-exporter-uaa-clients.yml) to your bosh director deployment

      bosh -d victoria-metrics deploy manifests/victoria-metrics.yml \
            -o manifests/operators/monitor-bosh.yml \
            -v bosh_url= \
            -o manifests/operators/enable-bosh-uaa.yml \
            -o manifests/operators/configure-bosh-exporter-uaa-client-id.yml \
            -v uaa_bosh_exporter_client_id= \
            -v uaa_bosh_exporter_client_secret= \
            --var-file bosh_ca_cert= \
            -v metrics_environment= \
            -o manifests/operators/monitor-cf.yml \
            -v uaa_clients_cf_exporter_id=cf_exporter \
            -v uaa_clients_cf_exporter_secret= \
            -v uaa_clients_firehose_exporter_id=firehose_exporter \
            -v uaa_clients_firehose_exporter_secret= \
            -v traffic_controller_external_port=443 \
            -v skip_ssl_verify=true \
            -o manifests/ops.yaml \
            -o manifests/operators/enable-cf-loggregator-v2.yml \
            -v metron_deployment_name=cf \
            -o manifests/operators/enable-cf-route-registrar.yml \
            -v cf_deployment_name= \
            -v system_domain=

* Please provide all required variables for command above.
* UAA clients for cf_exporter and firehose_exporter should be created with [manifests/operators/cf/add-cf-uaa-clients.yml ](https://github.com/VictoriaMetrics/victoriametrics-boshrelease/blob/master/manifests/operators/cf/add-cf-uaa-clients.yml) ops file applied to your CF deployment.


You can access grafana UI on https://grafana.system_domain
- Username - admin
- Password check in credhub with
```bash 
 credhub get --name /bosh/victoria-metrics/grafana_password
```

</details>
