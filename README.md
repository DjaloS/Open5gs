# Open5gs
Open5gs avec K8S 

# clonner le repo et deployé 

git clone https://github.com/DjaloS/Open5gs.git

cd Open5gs/helm-chart

helm -n open5gs install -f values.yaml open5gs ./

# pour voir la progression du deploiement 

kubectl -n open5gs get pods --watch


