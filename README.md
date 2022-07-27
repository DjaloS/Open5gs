# Open5gs
Open5gs on K8s 

# Deploy open5gs with helm-chart

git clone https://github.com/DjaloS/Open5gs.git

cd helm-chart

kubectl create ns open5gs

helm -n open5gs install -f values.yaml open5gs ./

# Make sure all POD & Services already running 

kubectl -n open5gs get pods --watch

# Configure Access to Open5Gs dashboard

cat <<EOF | sudo tee open5gs-webui-gateway.yaml

apiVersion: networking.istio.io/v1alpha3

kind: Gateway

metadata:
  name: open5gs-webui-gateway
spec:
  selector:
    istio: ingressgateway # use istio default controller
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "*"
    - 
---
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: open5gs-webui
spec:
  hosts:
  - "*"
  gateways:
  - open5gs-webui-gateway
  http:
  - match:
    route:
    - destination:
        host: open-webui.open5gs.svc.cluster.local
        port:
          number: 80
EOF

kubectl apply -f open5gs-webui-gateway.yamlNODE_PORT_OPEN5GS=$(kubectl -n istio-system get svc istio-ingressgateway \
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

