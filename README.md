
# Open5gs
Open5gs on K8s 

# Prérequis 
Install Service mesh Istio (optional)

curl -L https://istio.io/downloadIstio | ISTIO_VERSION=1.12.2  sh -

cd istio-1.12.2/

export PATH=$PWD/bin:$PATH

istioctl install --set profile=demo -y

Add a namespace label to instruct Istio to automatically inject Envoy sidecar proxies when you deploy your application later:

kubectl create ns open5gs

kubectl label namespace open5gs istio-injection=enabled

# Install Addons packages

cd ~/istio-1.12.2/samples/addons

kubectl apply -f prometheus.yaml #for data sources monitoring

kubectl apply -f grafana.yaml #for dashboard monitoring (visualization)

# Deploy open5gs with helm-chart

git clone https://github.com/DjaloS/Open5gs.git

cd helm-chart

kubectl create ns open5gs

helm -n open5gs install -f values.yaml open5gs ./

helm uninstall open -n open5gs

# Make sure all POD & Services already running 

kubectl -n open5gs get pods --watch

# Configure Access to Open5Gs dashboard

kubectl apply -f open5gs-webui-gateway.yaml

NODE_PORT_OPEN5GS=$(kubectl -n istio-system get svc istio-ingressgateway \
  --output=jsonpath='{range .spec.ports[1]}{.nodePort}')
  
  
echo $NODE_PORT_OPEN5GS

# Register User Equipment (UE) with detail bellow :

IMSI : 208930000000001

Key : 465B5CE8B199B49FAA5F0A2EE238A6BC

OP : E8ED289DEBA952E4283B54E88E6183CA

opType: OPC

apn: internet

sst: 1

sd: "ffffff"


# Configure UERANSIM (UE & gNB)

Install UERANSIM Helm depedency 

git clone https://github.com/DjaloS/openverso-charts.git

cd ~/openverso-charts/charts/ueransim

helm dep update ./


# check value in UE

sudo cat values.yaml

mcc: '208'

mnc: '93'

tac: '7'

# Change AMF Address in gNB

You must change address to AMF POD address, check with below command

kubectl  get  pod  -o  wide  -n open5gs  | grep amfAMF_POD_NAME= $(kubectl get pods  -o=name -n open5gs | grep  open-amf | awk -F"/" '{print $2}')AMF_ADDR=$( kubectl -n open5gs get pod $AMF_POD_NAME --template={{.status.podIP}})
echo ${AMF_ADDR}


sed -i "s/\${AMF_ADDR}/${AMF_ADDR}/g" resources/gnb.yaml


# verify with the command:

sudo cat resources/gnb.yaml


# Running UERANSIM

helm -n open5gs install -f values.yaml ueransim ./

kubectl get pod -n open5gs | grep ueransim

# Verify Logs UE Connected to gNB & AMF
check the AMF Logs with the command :

kubectl -n open5gs logs $AMF_POD_NAME

check the gNB Logs with the command :

kubectl -n open5gs logs ueransim-0 -c gnodeb

check the UE Logs with the command :

kubectl -n open5gs logs ueransim-0 -c ues

# Getting access to Grafana
Getting Grafana url

kubectl -n istio-system expose deployment grafana --port=3000  --name=grafana-http --type NodePort
NODE_PORT_GRAFANA=$(kubectl -n istio-system get svc grafana-http \
  --output=jsonpath='{range.spec.ports[0]}{.nodePort}')
echo $NODE_PORT_GRAFANA



