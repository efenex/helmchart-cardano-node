1. Install Prometheus

- Update lines from 2354 on values_cheap.yaml with static config for scrapping in format relay-{replica}.{releasename}.{namespace}.svc.cluster.local:{prometheusport}
- helm install stable/prometheus -f .\values_cheap.yaml -n monitoring

2. Install Grafana

- Create secret with kubectl for private & public key for SSL cert
- helm install grafana stable/grafana --set plugins=grafana-clock-panel,ingress.enabled=true,ingress.hosts[0]="monitoring.cheapstaking.com",ingress.tls[0].hosts[0]="monitoring.cheapstaking.com",ingress.tls[0].secretName=monitoring-cert,rbac.create=true,persistence.enabled=true,persistence.size=8Gi -n monitoring

3. Install Cardano Node Relays

- helm install relays --set cardanoNode.relayReplicaCount=3,cardanoNode.topologyConfigMap=topologyiohk.json . -n passive
- When at tip run helm upgrade as below to nodes to full topology
- helm upgrade relays --set cardanoNode.relayReplicaCount=3 . -n passive

4. Install Cardano Core / BlockMaker

- Register Stake Pool
- Copy kes.skey, vrf.skey & op.cert to secrets.yaml in base64 format
- helm install cheap --set coreOnlyRelease=true -n cheap
- secrets.yaml added with a helm hook, to managed with kubectl going forward so releases cant overwrite other pools in cluster when upgrades are run.
