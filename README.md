
# Open5gs sur K8s à l'aide de Vagrant 

Avant tout d'abord il est imperatif de créer le repertoir où persister les donner mongo dans mon cas : /home/vagrant/kubedata dans le worker3

```shell
Créer un namespace
kubectl create ns open5gs
```

# Deployer open5gs avec helm-chart
```shell
git clone https://github.com/DjaloS/Open5gs.git
cd Open5gs/helm-chart
helm -n open5gs install -f values.yaml open5gs ./
```
# On s'assure que les pod sont en running avec la commande
```shell
kubectl -n open5gs get pods --watch
```
# Enregistrer l'équipement utilisateur (UE) avec les détails ci-dessous 
D'abord recuperer le port sur lequel il est disponible open-webui est disponible : voir l'image ci-dessous 

![image](https://user-images.githubusercontent.com/109952373/191764993-281bf8a4-d289-4e87-82c5-a93ae4737260.png)

comme on peut le voir sur la capture, l'interface est exposé sur le port: 31659

Se connecter sur le dashboard du open-webui en caisissant l'IP de la machine qur laquelle le pod open-webui tourne sur le navigateur: 

![image](https://user-images.githubusercontent.com/109952373/191766926-9ef25813-ad62-4202-a857-2ffef0d75711.png)

comme on peut le voit sur la capture, open-webui tourne sur le master donc on saisi : **http://192.168.56.10:31659/**

Pour se connecté sur l'interface:

``shell
Username : admin
Password : 1423
``
![image](https://user-images.githubusercontent.com/109952373/191768716-96f726ba-c12a-4ef6-88bb-a899d16191c0.png)

Une fois connecté, nous allons proceder à la configuration du User Equipment (UE) avec les infos ci-dessous 

```shell
IMSI : 208930000000001
Key : 465B5CE8B199B49FAA5F0A2EE238A6BC
OP : E8ED289DEBA952E4283B54E88E6183CA
opType: OPC
apn: internet
sst: 1
sd: "ffffff"
```

# Configuration du Réseau d'accès (UE & gNB)
```shell
git clone https://github.com/DjaloS/openverso-charts.git
cd ~/openverso-charts/charts/ueransim
helm dep update ./
```
# Verification des valeurs mcc, mnc et tac dans L'UE:
```shell
sudo cat values.yaml
mcc: '208'  ## Le code du pay
mnc: '93'   ## le code de l'operateur
tac: '7'
```
# Configurer l'IP de l'AMF dans le gNB

on place le nom du pod amf dans une variable pour le mettre dans le gnb de sort que le gnb ait la même IP
```shell
kubectl get pod -o wide -n open5gs | grep amf

AMF_POD_NAME=$(kubectl get pods -o=name -n open5gs | grep  open-amf | awk -F"/" '{print $2}')  

AMF_ADDR=$(kubectl -n open5gs get pod $AMF_POD_NAME --template={{.status.podIP}})
echo ${AMF_ADDR}
192.16.219.68                               

![image](https://user-images.githubusercontent.com/109952373/191769950-685ebb57-7b1e-4180-a924-6527b47c75fb.png)

IL doit vous retourner l'IP du l'AMF

sed -i "s/\${AMF_ADDR}/${AMF_ADDR}/g" resources/gnb.yaml
```

# verification de la config avec la commande:
```sehll
sudo cat resources/gnb.yaml
```
# Installation de L'UERANSIM
```shell
helm -n open5gs install -f values.yaml ueransim ./
kubectl get pod -n open5gs | grep ueransim
```
# Verification de la connectivité du gNB & AMF à travers les logs

```shell
verifier les logs AMF avec la commande

kubectl -n open5gs logs $AMF_POD_NAME
```
![image](https://user-images.githubusercontent.com/109952373/191771985-86a75a88-d2d6-4c17-b7bf-8bbc7b25362d.png)


verifier les logs du gNB avec la commande
```shell
kubectl -n open5gs logs ueransim-0 -c gnodeb
```
![image](https://user-images.githubusercontent.com/109952373/191772313-3494a1ca-4191-4943-8ef8-dce56a434aff.png)

verifier les logs du l'UE avec la commande
```shell
kubectl -n open5gs logs ueransim-0 -c ues
```
![image](https://user-images.githubusercontent.com/109952373/191772792-39eb8797-c3d4-46c7-b9ba-466d3c7b83f8.png)





