# Monitoring-With-Prometheus-and-Grafana
Monitoring With Prometheus and Grafana


Install Prometheus and Grafana
Add the Helm repos for Prometheus, Grafana and update any changes.



helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
We’ll create a separate namespace for Prometheus and install the chart there.



kubectl create namespace prometheus
helm install prometheus prometheus-community/prometheus \
    --namespace prometheus \
    --set alertmanager.persistentVolume.storageClass="gp2" \
    --set server.persistentVolume.storageClass="gp2"
Create this file. It tells Grafana what to use as a source (url). It’s the Prometheus endpoint service that we just installed.



cat << EOF > grafana.yaml
datasources:
  datasources.yaml:
    apiVersion: 1
    datasources:
    - name: Prometheus
      type: prometheus
      url: http://prometheus-server.prometheus.svc.cluster.local
      access: proxy
      isDefault: true
EOF
Now, let’s create a namespace for Grafana and then install Grafana in that namespace. Change the admin password. In my case it’s Password1$.



kubectl create namespace grafana
helm install grafana grafana/grafana \
    --namespace grafana \
    --set persistence.storageClassName="gp2" \
    --set persistence.enabled=true \
    --set adminPassword='Password1$' \
    --values grafana.yaml \
    --set service.type=LoadBalancer
You’ll get something like this on the screen when the install ends.


export SERVICE_IP=$(kubectl get svc --namespace grafana grafana -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
That’s your load balancer IP so you can access Grafana. If you use EKS, your IP is different.


kubectl get services -n grafana
NAME      TYPE           CLUSTER-IP     EXTERNAL-IP                                                               PORT(S)        AGE
grafana   LoadBalancer   10.100.20.53   a074996f790b74793a74bec584c04460-2034633908.us-east-2.elb.amazonaws.com   80:31916/TCP   3m21s
The IP is the gibberish URL that ends with amazonaws.com. If you go to that URL, you can log as admin and your password.

Go to Dashboards, click on the New button on the right and click Import.
Add the following dashboard IDs (one by one then click Load) and choose Prometheus as a source. The IDs are 3119 (Kubernetes cluster), 6417 (pods) and 11159 (Node.js). For the first dashboard, you’ll have some metrics. The Node.js dashboard is empty and our goal is to get the metrics from the application.

REF: https://blog.andreev.it/2023/01/kubernetes-monitoring-node-js-application-with-prometheus-and-grafana-helm-charts/

