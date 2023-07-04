# Victoria Metrics BOSH release 
This bosh release helps you to deploy victoria-metrics tsdb and provide monitoring solution for your bosh managed environment.
## Prerequisites:
1. bosh environment
2. bosh upload-release 
3. bosh stemcells
4. Able to resolve cf.internal domains to use routing release. You can apply 
```
bosh update-runtime-config --name cf-alias manifests/operators/bosh/bosh-alias-runtime.yaml
```

Examples are made with bosh-bootloader and credhub.

## Victoria-metrics deployment cases:
<details>
      <summary>Standalone victoria-metrics tsdb with basic auth and tls without bosh integration</summary>
               
      bosh -d victoria-metrics deploy manifests/victoria-metrics.yml \
            -o manifests/operators/enable-tls.yml \
            -v domain= \
            -v skip_ssl_verify=true \
            -o manifests/operators/singlevm.yml

* You can use [manifests/operators/gcp_ops.yaml ](https://github.com/VictoriaMetrics/victoriametrics-boshrelease/blob/master/manifests/operators/gcp_ops.yaml) as an example to adjust bosh cloud-config
* domain - wildcard domain added to tls certificate for victoria-metrics UI
      
      bosh -d victoria-metrics deploy manifests/victoria-metrics.yml \
      -o manifests/operators/gcp_ops.yaml \
      -o manifests/operators/enable-tls.yml \
      -v domain= \
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


* Please provide all required variables for command above.
      * bosh_url - bosh director ip address, example 10.0.0.6
      * uaa_bosh_exporter_client_id and secret - Create UAA client for bosh-exporter by applying [manifests/operators/bosh/add-bosh-exporter-uaa-clients.yml ](https://github.com/VictoriaMetrics/victoriametrics-boshrelease/blob/master/manifests/operators/bosh/add-bosh-exporter-uaa-clients.yml) to your bosh director deployment. 
            client_id - ```bosh_exporter```, secret ```bosh int vars/director-vars-store.yml --path=/uaa_bosh_exporter_client_secret --json | jq -rj .Blocks[]``` for secrets stored in vars.
      * bosh_ca_cert - path to bosh CA certificate file. If bbl was used -```eval "$(bbl print-env)" && bbl director-ca-cert > root.ca```
      * metrics_environment - label for your environment, ex. lab
      * uaa_clients_cf_exporter_id/secret and uaa_clients_firehose_exporter_id/secret - UAA clients for cf_exporter and firehose_exporter should be created with [manifests/operators/cf/add-cf-uaa-clients.yml ](https://github.com/VictoriaMetrics/victoriametrics-boshrelease/blob/master/manifests/operators/cf/add-cf-exporter-uaa-clients.yml) ops file applied to your CF deployment. Find secrets with ```credhub find -n cf_exporter``` and ```credhub find -n firehose_exporter```
      * cf_deployment_name - cloudfoundry deployment name. Example cf.
      * system_domain - cloudfoundry system domain. Example sys.example.com
      * nats_certificate and nats_private_key - paths to credhub. Use ```credhub find -n nats_client_cert```


      bosh -d victoria-metrics deploy manifests/victoria-metrics.yml \
            -o manifests/operators/monitor-bosh.yml \
            -v bosh_url= \
            -o manifests/operators/enable-bosh-uaa.yml \
            -o manifests/operators/configure-bosh-exporter-uaa-client-id.yml \
            -v uaa_bosh_exporter_client_id= \
            -v uaa_bosh_exporter_client_secret= \
            --var-file bosh_ca_cert= \
            -v metrics_environment=lab \
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
            -v system_domain= \
            -v nats_certificate= \
            -v nats_private_key=


You can access grafana UI on https://grafana.system_domain
- Username - admin
- Password check in credhub with
```bash 
 credhub get --name /bosh/victoria-metrics/grafana_password
```

</details>
