# Open5gs
Open5gs on K8s 

# Deploy open5gs with helm-chart

git clone https://github.com/DjaloS/Open5gs.git

cd helm-chart

kubectl create ns open5gs

helm -n open5gs install -f values.yaml open5gs ./
