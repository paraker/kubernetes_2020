# Create deployment to use hpa on
kubectl create -f https://raw.githubusercontent.com/paraker/kubernetes_2020/master/ckad/hpa/apache_deployment.yaml

# Create hpa
kubectl create -f https://raw.githubusercontent.com/paraker/kubernetes_2020/master/ckad/hpa/apache_hpa.yaml

(or run kubectl autoscale deployment php-apache --cpu-percent=50 --min=1 --max=10)

# increase the load on apache
kubectl get hpa
kubectl run -i --tty load-generator --rm --image=busybox --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://php-apache; done"
kubectl get hpa
kubectl get deployment php-apache
