
# Install

1. HTTP負荷分散 無効
2. load-balancer ノードプール作成
gcloud container clusters get-credentials main-03 --zone us-central1-a
gcloud compute addresses create main-03-ip --region us-central1
export LB_ADDRESS_IP=$(gcloud compute addresses list | grep main-03-ip | awk '{print $3}')
export LB_INSTANCE_NAME=$(kubectl describe nodes | grep Name: | tail -n1 | awk '{print $2}')
export LB_INSTANCE_NAT=external-nat
kubectl taint nodes $LB_INSTANCE_NAME role=nginx-ingress-controller:NoSchedule
gcloud compute instances delete-access-config $LB_INSTANCE_NAME \
 --access-config-name "$LB_INSTANCE_NAT"
gcloud compute instances add-access-config $LB_INSTANCE_NAME \
 --access-config-name "$LB_INSTANCE_NAT" --address $LB_ADDRESS_IP
kubectl label nodes $LB_INSTANCE_NAME role=load-balancer
gcloud compute instances add-tags $LB_INSTANCE_NAME --tags http-server,https-server
helm init --service-account tiller
echo "apiVersion: v1
kind: ServiceAccount
metadata:
  name: tiller
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: tiller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: tiller
    namespace: kube-system
" | kubectl apply -f -

cd nginx-ingress
eval `cat cmd.txt`
cd ../
kubectl apply -f nginx-ingress.yml
cd $HOME/cert-manager
 helm install \
    --name cert-manager \
    --namespace=kube-system \
    contrib/charts/cert-manager \
    --set ingressShim.extraArgs='{--default-issuer-name=letsencrypt-prod,--default-issuer-kind=ClusterIssuer}'

#kubectl create secret generic route53-config --from-file=./secretaccesskey.txt -n kube-system
cd $HOME/root/personal-project/nginx-ingress
kubectl apply -f ClusterIssuer.yml
kubectl apply -f certificate.yml
kubectl describe certificate
# kubectl get secret route53-config -o=go-template --template '{{index .data "secretaccesskey.txt"}}' | base64 -d
cd $HOME/keel
 helm install \
    --name keel \
    --namespace=kube-system \
    chart/keel \
    -f $HOME/root/personal-project/keel/values.yaml
