
# Open5gs
Open5gs on K8s 
# Créer un namespace

kubectl create ns open5gs

# Deployer open5gs avec helm-chart

git clone https://github.com/DjaloS/Open5gs.git

cd helm-chart

helm -n open5gs install -f values.yaml open5gs ./

# Make sure all POD & Services already running 

kubectl -n open5gs get pods --watch

# Register User Equipment (UE) with detail bellow :

IMSI : 208930000000001

Key : 465B5CE8B199B49FAA5F0A2EE238A6BC

OP : E8ED289DEBA952E4283B54E88E6183CA

opType: OPC

apn: internet

sst: 1

sd: "ffffff"

# Configuration du Réseau d'accès (UE & gNB)

git clone https://github.com/DjaloS/openverso-charts.git

cd ~/openverso-charts/charts/ueransim

helm dep update ./

# Verification des valeurs mcc, mnc et tac:

sudo cat values.yaml

mcc: '208'

mnc: '93'

tac: '7'

# Preciser l'IP du pod AMF dans le gNB

kubectl  get  pod  -o  wide  -n open5gs  | grep amfAMF_POD_NAME= $(kubectl get pods -o=name  -n  open5gs | grep open-amf | awk -F "/"  '{print $2}')AMF_ADDR=$( kubectl   -n  open5gs get pod $AMF_POD_NAME  --template={{.status.podIP}})
echo ${AMF_ADDR}

sed -i "s/\${AMF_ADDR}/${AMF_ADDR}/g" resources/gnb.yaml

# verifier avec la commande:

sudo cat resources/gnb.yaml

# Installation de L'UERANSIM

helm -n open5gs install -f values.yaml ueransim ./

kubectl get pod -n open5gs | grep ueransim

# Verify Logs UE Connected to gNB & AMF
check the AMF Logs with the command :

kubectl -n open5gs logs $AMF_POD_NAME

check the gNB Logs with the command :

kubectl -n open5gs logs ueransim-0 -c gnodeb

check the UE Logs with the command :

kubectl -n open5gs logs ueransim-0 -c ues






