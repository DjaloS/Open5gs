
# Open5gs sur K8s à l'aide de Vagrant 

**Avant tout d'abord il est imperatif de créer le repertoir où persister les donner mongo dans mon cas : /home/vagrant/kubedata**

 

**Créer un namespace**
```shell

kubectl create ns open5gs

# Deployer open5gs avec helm-chart
```shell
git clone https://github.com/DjaloS/Open5gs.git
cd helm-chart
helm -n open5gs install -f values.yaml open5gs ./

# Assurez vous que les pod sont en running 

kubectl -n open5gs get pods --watch

```shell
Enregistrer l'équipement utilisateur (UE) avec les détails ci-dessous 

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

# Verification des valeurs mcc, mnc et tac dans L'UE:

sudo cat values.yaml
```shell
mcc: '208'  ## Le du pay
mnc: '93'   ## le code de l'operateur
tac: '7'

# Preciser l'IP du pod AMF dans le gNB

```shell
kubectl  get  pod  -o  wide  -n open5gs  | grep amfAMF_POD_NAME= $(kubectl get pods -o=name  -n  open5gs | grep open-amf | awk -F"/" '{print $2}')AMF_ADDR=$( kubectl   -n  open5gs get pod $AMF_POD_NAME  --template={{.status.podIP}})
echo ${AMF_ADDR}
sed -i "s/\${AMF_ADDR}/${AMF_ADDR}/g" resources/gnb.yaml

# verifier avec la commande:
```shell
sudo cat resources/gnb.yaml

# Installation de L'UERANSIM
```shell
helm -n open5gs install -f values.yaml ueransim ./
kubectl get pod -n open5gs | grep ueransim

# Verifier à taravers les Logs que l'UE et Connecter au gNB & AMF

**verifier les logs AMF avec la commande :**
```shell
kubectl -n open5gs logs $AMF_POD_NAME

**verifier les logs du gNB avec la commande :**
```shell
kubectl -n open5gs logs ueransim-0 -c gnodeb
```shell
**verifier les logs du l'UE avec la commande :**
```shell
kubectl -n open5gs logs ueransim-0 -c ues






