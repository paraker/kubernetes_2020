# Create deployment to use hpa on
kubectl create -f https://raw.githubusercontent.com/paraker/kubernetes_2020/master/ckad/hpa/apache_deployment.yaml

# Create hpa
kubectl create -f https://raw.githubusercontent.com/paraker/kubernetes_2020/master/ckad/hpa/apache_hpa.yaml

# create cron job for hpa
kubectl create -f https://raw.githubusercontent.com/paraker/kubernetes_2020/master/ckad/hpa/cron_hpa.yaml


# kubectl commands
`kubectl autoscale deployment php-apache --cpu-percent=50 --min=1 --max=10`

`kubectl patch hpa php-apache -p '{"spec":{"minReplicas":7}}'` # patch to a set number of minimum replicas

# increase the load on apache
kubectl get hpa
kubectl run -i --tty load-generator --rm --image=busybox --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://php-apache; done"
kubectl get hpa
kubectl get deployment php-apache
