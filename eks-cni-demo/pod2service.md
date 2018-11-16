# Pod to Service Demo

[Detailed Info](https://kubernetes.io/docs/tasks/access-application-cluster/connecting-frontend-backend/)

## Creating the backend using a Deployment

```
kubectl apply -f hello.yaml
```

```
kubectl get deployment hello
NAME      DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
hello     7         7         7            7           50s
```

```
kubectl describe deployment hello
Name:                   hello
Namespace:              default
CreationTimestamp:      Wed, 14 Nov 2018 01:30:29 +0000
Labels:                 <none>
Annotations:            deployment.kubernetes.io/revision=1
                        kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"apps/v1","kind":"Deployment","metadata":{"annotations":{},"name":"hello","namespace":"default"},"spec":{"replicas":7,"selector":{"matchL...
Selector:               app=hello,tier=backend,track=stable
Replicas:               7 desired | 7 updated | 7 total | 7 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=hello
           tier=backend
           track=stable
  Containers:
   hello:
    Image:        gcr.io/google-samples/hello-go-gke:1.0
    Port:         80/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   hello-7ff54bc875 (7/7 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  1m    deployment-controller  Scaled up replica set hello-7ff54bc875 to 7

```

## Creating the backend Service Object (ClusterIP, kube-proxy)

```
kubectl apply -f hello-service.yaml 
```

```
kubectl get service
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
hello        ClusterIP   10.100.204.119   <none>        80/TCP    37s
kubernetes   ClusterIP   10.100.0.1       <none>        443/TCP   2h  
```

```
kubectl describe service hello
Name:              hello
Namespace:         default
Labels:            <none>
Annotations:       kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"name":"hello","namespace":"default"},"spec":{"ports":[{"port":80,"protocol":"TCP","ta...
Selector:          app=hello,tier=backend
Type:              ClusterIP
IP:                10.100.204.119
Port:              <unset>  80/TCP
TargetPort:        http/TCP
Endpoints:         192.168.136.140:80,192.168.149.34:80,192.168.187.204:80 + 4 more...
Session Affinity:  None
Events:            <none>

```

### Exam IPtables (Pod -> ClusterIP(10.100.204.119) -> Endpoints (192.168.136.140, 192.168.149.34, ...))

```
ssh -i <key-name> ec2-user@<worker-IP>
```

#### Display IPtables
```
sudo iptables-save

# Generated by iptables-save v1.4.21 on Wed Nov 14 01:43:10 2018
*mangle
:PREROUTING ACCEPT [123097:69451088]
:INPUT ACCEPT [102684:63542421]
:FORWARD ACCEPT [20413:5908667]
:OUTPUT ACCEPT [88691:7509314]
:POSTROUTING ACCEPT [109104:13417981]
-A PREROUTING -i eth0 -m comment --comment "AWS, primary ENI" -m addrtype --dst-type LOCAL --limit-iface-in -j CONNMARK --set-xmark 0x80/0x80
-A PREROUTING -i eni+ -m comment --comment "AWS, primary ENI" -j CONNMARK --restore-mark --nfmask 0x80 --ctmask 0x80
COMMIT
# Completed on Wed Nov 14 01:43:10 2018
# Generated by iptables-save v1.4.21 on Wed Nov 14 01:43:10 2018
*nat
:PREROUTING ACCEPT [1:64]
:INPUT ACCEPT [1:64]
:OUTPUT ACCEPT [33:1994]
:POSTROUTING ACCEPT [10:614]
:DOCKER - [0:0]
:KUBE-MARK-DROP - [0:0]
:KUBE-MARK-MASQ - [0:0]
:KUBE-NODEPORTS - [0:0]
:KUBE-POSTROUTING - [0:0]
:KUBE-SEP-5FZSVYKRYDUCHFON - [0:0]
:KUBE-SEP-AIEIM4BQDHX5EYIM - [0:0]
:KUBE-SEP-ASC4DHGHNRBGNQB4 - [0:0]
:KUBE-SEP-B335W6MFC4JCQI26 - [0:0]
:KUBE-SEP-CNWBF4XGFTZ6O3ZN - [0:0]
:KUBE-SEP-GFRP3FYZNBPEDFKH - [0:0]
:KUBE-SEP-PC6PCESA3SOZ3W23 - [0:0]
:KUBE-SEP-VMSA6O6SI2ZXLCX3 - [0:0]
:KUBE-SEP-VXYJY4HL2DUWJW4T - [0:0]
:KUBE-SEP-YAHRLKRDJOGGDZAI - [0:0]
:KUBE-SEP-ZXNKJZGHRUCBYABC - [0:0]
:KUBE-SERVICES - [0:0]
:KUBE-SVC-ERIFXISQEP7F7OF4 - [0:0]
:KUBE-SVC-NPX46M4PTMTKRN6Y - [0:0]
:KUBE-SVC-TCOU7JCQXEZGVUNU - [0:0]
:KUBE-SVC-TWVLBX4WCEZSIVWL - [0:0]
-A PREROUTING -m comment --comment "kubernetes service portals" -j KUBE-SERVICES
-A PREROUTING -m addrtype --dst-type LOCAL -j DOCKER
-A OUTPUT -m comment --comment "kubernetes service portals" -j KUBE-SERVICES
-A OUTPUT ! -d 127.0.0.0/8 -m addrtype --dst-type LOCAL -j DOCKER
-A POSTROUTING -m comment --comment "kubernetes postrouting rules" -j KUBE-POSTROUTING
-A POSTROUTING -s 172.17.0.0/16 ! -o docker0 -j MASQUERADE
-A POSTROUTING ! -d 192.168.0.0/16 -m comment --comment "AWS, SNAT" -m addrtype ! --dst-type LOCAL -j SNAT --to-source 192.168.182.224
-A DOCKER -i docker0 -j RETURN
-A KUBE-MARK-DROP -j MARK --set-xmark 0x8000/0x8000
-A KUBE-MARK-MASQ -j MARK --set-xmark 0x4000/0x4000
-A KUBE-POSTROUTING -m comment --comment "kubernetes service traffic requiring SNAT" -m mark --mark 0x4000/0x4000 -j MASQUERADE
-A KUBE-SEP-5FZSVYKRYDUCHFON -s 192.168.74.164/32 -m comment --comment "default/kubernetes:https" -j KUBE-MARK-MASQ
-A KUBE-SEP-5FZSVYKRYDUCHFON -p tcp -m comment --comment "default/kubernetes:https" -m recent --set --name KUBE-SEP-5FZSVYKRYDUCHFON --mask 255.255.255.255 --rsource -m tcp -j DNAT --to-destination 192.168.74.164:443
-A KUBE-SEP-AIEIM4BQDHX5EYIM -s 192.168.128.239/32 -m comment --comment "default/kubernetes:https" -j KUBE-MARK-MASQ
-A KUBE-SEP-AIEIM4BQDHX5EYIM -p tcp -m comment --comment "default/kubernetes:https" -m recent --set --name KUBE-SEP-AIEIM4BQDHX5EYIM --mask 255.255.255.255 --rsource -m tcp -j DNAT --to-destination 192.168.128.239:443
-A KUBE-SEP-ASC4DHGHNRBGNQB4 -s 192.168.149.34/32 -m comment --comment "default/hello:" -j KUBE-MARK-MASQ
-A KUBE-SEP-ASC4DHGHNRBGNQB4 -p tcp -m comment --comment "default/hello:" -m tcp -j DNAT --to-destination 192.168.149.34:80
-A KUBE-SEP-B335W6MFC4JCQI26 -s 192.168.179.184/32 -m comment --comment "kube-system/kube-dns:dns-tcp" -j KUBE-MARK-MASQ
-A KUBE-SEP-B335W6MFC4JCQI26 -p tcp -m comment --comment "kube-system/kube-dns:dns-tcp" -m tcp -j DNAT --to-destination 192.168.179.184:53
-A KUBE-SEP-CNWBF4XGFTZ6O3ZN -s 192.168.187.204/32 -m comment --comment "default/hello:" -j KUBE-MARK-MASQ
-A KUBE-SEP-CNWBF4XGFTZ6O3ZN -p tcp -m comment --comment "default/hello:" -m tcp -j DNAT --to-destination 192.168.187.204:80
-A KUBE-SEP-GFRP3FYZNBPEDFKH -s 192.168.229.17/32 -m comment --comment "default/hello:" -j KUBE-MARK-MASQ
-A KUBE-SEP-GFRP3FYZNBPEDFKH -p tcp -m comment --comment "default/hello:" -m tcp -j DNAT --to-destination 192.168.229.17:80
-A KUBE-SEP-PC6PCESA3SOZ3W23 -s 192.168.216.216/32 -m comment --comment "default/hello:" -j KUBE-MARK-MASQ
-A KUBE-SEP-PC6PCESA3SOZ3W23 -p tcp -m comment --comment "default/hello:" -m tcp -j DNAT --to-destination 192.168.216.216:80
-A KUBE-SEP-VMSA6O6SI2ZXLCX3 -s 192.168.179.184/32 -m comment --comment "kube-system/kube-dns:dns" -j KUBE-MARK-MASQ
-A KUBE-SEP-VMSA6O6SI2ZXLCX3 -p udp -m comment --comment "kube-system/kube-dns:dns" -m udp -j DNAT --to-destination 192.168.179.184:53
-A KUBE-SEP-VXYJY4HL2DUWJW4T -s 192.168.136.140/32 -m comment --comment "default/hello:" -j KUBE-MARK-MASQ
-A KUBE-SEP-VXYJY4HL2DUWJW4T -p tcp -m comment --comment "default/hello:" -m tcp -j DNAT --to-destination 192.168.136.140:80
-A KUBE-SEP-YAHRLKRDJOGGDZAI -s 192.168.207.166/32 -m comment --comment "default/hello:" -j KUBE-MARK-MASQ
-A KUBE-SEP-YAHRLKRDJOGGDZAI -p tcp -m comment --comment "default/hello:" -m tcp -j DNAT --to-destination 192.168.207.166:80
-A KUBE-SEP-ZXNKJZGHRUCBYABC -s 192.168.244.95/32 -m comment --comment "default/hello:" -j KUBE-MARK-MASQ
-A KUBE-SEP-ZXNKJZGHRUCBYABC -p tcp -m comment --comment "default/hello:" -m tcp -j DNAT --to-destination 192.168.244.95:80
-A KUBE-SERVICES -d 10.100.0.10/32 -p udp -m comment --comment "kube-system/kube-dns:dns cluster IP" -m udp --dport 53 -j KUBE-SVC-TCOU7JCQXEZGVUNU
-A KUBE-SERVICES -d 10.100.0.10/32 -p tcp -m comment --comment "kube-system/kube-dns:dns-tcp cluster IP" -m tcp --dport 53 -j KUBE-SVC-ERIFXISQEP7F7OF4
-A KUBE-SERVICES -d 10.100.0.1/32 -p tcp -m comment --comment "default/kubernetes:https cluster IP" -m tcp --dport 443 -j KUBE-SVC-NPX46M4PTMTKRN6Y
-A KUBE-SERVICES -d 10.100.204.119/32 -p tcp -m comment --comment "default/hello: cluster IP" -m tcp --dport 80 -j KUBE-SVC-TWVLBX4WCEZSIVWL
-A KUBE-SERVICES -m comment --comment "kubernetes service nodeports; NOTE: this must be the last rule in this chain" -m addrtype --dst-type LOCAL -j KUBE-NODEPORTS
-A KUBE-SVC-ERIFXISQEP7F7OF4 -m comment --comment "kube-system/kube-dns:dns-tcp" -j KUBE-SEP-B335W6MFC4JCQI26
-A KUBE-SVC-NPX46M4PTMTKRN6Y -m comment --comment "default/kubernetes:https" -m recent --rcheck --seconds 10800 --reap --name KUBE-SEP-AIEIM4BQDHX5EYIM --mask 255.255.255.255 --rsource -j KUBE-SEP-AIEIM4BQDHX5EYIM
-A KUBE-SVC-NPX46M4PTMTKRN6Y -m comment --comment "default/kubernetes:https" -m recent --rcheck --seconds 10800 --reap --name KUBE-SEP-5FZSVYKRYDUCHFON --mask 255.255.255.255 --rsource -j KUBE-SEP-5FZSVYKRYDUCHFON
-A KUBE-SVC-NPX46M4PTMTKRN6Y -m comment --comment "default/kubernetes:https" -m statistic --mode random --probability 0.50000000000 -j KUBE-SEP-AIEIM4BQDHX5EYIM
-A KUBE-SVC-NPX46M4PTMTKRN6Y -m comment --comment "default/kubernetes:https" -j KUBE-SEP-5FZSVYKRYDUCHFON
-A KUBE-SVC-TCOU7JCQXEZGVUNU -m comment --comment "kube-system/kube-dns:dns" -j KUBE-SEP-VMSA6O6SI2ZXLCX3
-A KUBE-SVC-TWVLBX4WCEZSIVWL -m comment --comment "default/hello:" -m statistic --mode random --probability 0.14286000002 -j KUBE-SEP-VXYJY4HL2DUWJW4T
-A KUBE-SVC-TWVLBX4WCEZSIVWL -m comment --comment "default/hello:" -m statistic --mode random --probability 0.16667000018 -j KUBE-SEP-ASC4DHGHNRBGNQB4
-A KUBE-SVC-TWVLBX4WCEZSIVWL -m comment --comment "default/hello:" -m statistic --mode random --probability 0.20000000019 -j KUBE-SEP-CNWBF4XGFTZ6O3ZN
-A KUBE-SVC-TWVLBX4WCEZSIVWL -m comment --comment "default/hello:" -m statistic --mode random --probability 0.25000000000 -j KUBE-SEP-YAHRLKRDJOGGDZAI
-A KUBE-SVC-TWVLBX4WCEZSIVWL -m comment --comment "default/hello:" -m statistic --mode random --probability 0.33332999982 -j KUBE-SEP-PC6PCESA3SOZ3W23
-A KUBE-SVC-TWVLBX4WCEZSIVWL -m comment --comment "default/hello:" -m statistic --mode random --probability 0.50000000000 -j KUBE-SEP-GFRP3FYZNBPEDFKH
-A KUBE-SVC-TWVLBX4WCEZSIVWL -m comment --comment "default/hello:" -j KUBE-SEP-ZXNKJZGHRUCBYABC
COMMIT
# Completed on Wed Nov 14 01:43:10 2018
# Generated by iptables-save v1.4.21 on Wed Nov 14 01:43:10 2018
*filter
:INPUT ACCEPT [322:69320]
:FORWARD ACCEPT [44:14588]
:OUTPUT ACCEPT [277:29413]
:KUBE-EXTERNAL-SERVICES - [0:0]
:KUBE-FIREWALL - [0:0]
:KUBE-FORWARD - [0:0]
:KUBE-SERVICES - [0:0]
-A INPUT -m conntrack --ctstate NEW -m comment --comment "kubernetes externally-visible service portals" -j KUBE-EXTERNAL-SERVICES
-A INPUT -j KUBE-FIREWALL
-A FORWARD -m comment --comment "kubernetes forwarding rules" -j KUBE-FORWARD
-A OUTPUT -m conntrack --ctstate NEW -m comment --comment "kubernetes service portals" -j KUBE-SERVICES
-A OUTPUT -j KUBE-FIREWALL
-A KUBE-FIREWALL -m comment --comment "kubernetes firewall for dropping marked packets" -m mark --mark 0x8000/0x8000 -j DROP
-A KUBE-FORWARD -m comment --comment "kubernetes forwarding rules" -m mark --mark 0x4000/0x4000 -j ACCEPT
COMMIT
# Completed on Wed Nov 14 01:43:10 2018

```
  
  
## Creating the frontend (using AWS ELB)

```
kubectl apply -f frontend-elb.yaml
```

### frontend-elb.yaml

```
apiVersion: v1
kind: Service
metadata:
  name: frontend
spec:
  selector:
    app: hello
    tier: frontend
  ports:
  - protocol: "TCP"
    port: 80
    targetPort: 80
  type: LoadBalancer
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
spec:
  selector:
    matchLabels:
      app: hello
      tier: frontend
      track: stable
  replicas: 1
  template:
    metadata:
      labels:
        app: hello
        tier: frontend
        track: stable
    spec:
      containers:
      - name: nginx
        image: "gcr.io/google-samples/hello-frontend:1.0"
        lifecycle:
          preStop:
            exec:
              command: ["/usr/sbin/nginx","-s","quit"]
```

### kubectl describe service frontend

```
kubectl describe service frontend
Name:                     frontend
Namespace:                default
Labels:                   <none>
Annotations:              kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"name":"frontend","namespace":"default"},"spec":{"ports":[{"port":80,"protocol":"TCP",...
Selector:                 app=hello,tier=frontend
Type:                     LoadBalancer
IP:                       10.100.125.32
LoadBalancer Ingress:     aba776baae7b111e8830f02c004722d8-1367000266.us-west-2.elb.amazonaws.com
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  32668/TCP
Endpoints:                192.168.168.69:80
Session Affinity:         None
External Traffic Policy:  Cluster
Events:
  Type    Reason                Age   From                Message
  ----    ------                ----  ----                -------
  Normal  EnsuringLoadBalancer  21m   service-controller  Ensuring load balancer
  Normal  EnsuredLoadBalancer   21m   service-controller  Ensured load balancer
  
  
```

### Send traffic through the frontend (ELB)

```
curl aba776baae7b111e8830f02c004722d8-1367000266.us-west-2.elb.amazonaws.com
{"message":"Hello"}
```

## Create frontend using NLB
  
```
kubectl apply -f frontend-nlb.yaml
```

### frontend-nlb.yaml

```
apiVersion: v1
kind: Service
metadata:
  name: frontend-nlb
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-type: "nlb"  <--- NLB annotation
spec:
  selector:
    app: hello
    tier: frontend-nlb
  ports:
  - protocol: "TCP"
    port: 80
    targetPort: 80
  type: LoadBalancer
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-nlb
spec:
  selector:
    matchLabels:
      app: hello
      tier: frontend-nlb
      track: stable
  replicas: 1
  template:
    metadata:
      labels:
        app: hello
        tier: frontend-nlb
        track: stable
    spec:
      containers:
      - name: nginx
        image: "gcr.io/google-samples/hello-frontend:1.0"
        lifecycle:
          preStop:
            exec:
              command: ["/usr/sbin/nginx","-s","quit"]
              
```     

### kubectl describe service frontend-nlb

```
kubectl describe svc frontend-nlb
Name:                     frontend-nlb
Namespace:                default
Labels:                   <none>
Annotations:              kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"v1","kind":"Service","metadata":{"annotations":{"service.beta.kubernetes.io/aws-load-balancer-type":"nlb"},"name":"frontend-nlb","namesp...
                          service.beta.kubernetes.io/aws-load-balancer-type=nlb
Selector:                 app=hello,tier=frontend-nlb
Type:                     LoadBalancer
IP:                       10.100.190.43
LoadBalancer Ingress:     a5b0fe0e6e7ba11e88c030671ac002d2-586d77860e47b4f5.elb.us-west-2.amazonaws.com
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  31034/TCP
Endpoints:                192.168.213.7:80
Session Affinity:         None
External Traffic Policy:  Cluster
Events:
  Type    Reason                Age   From                Message
  ----    ------                ----  ----                -------
  Normal  EnsuringLoadBalancer  3m    service-controller  Ensuring load balancer
  Normal  EnsuredLoadBalancer   3m    service-controller  Ensured load balancer
           
```

### Send traffic through the frontend (NLB)

```
a5b0fe0e6e7ba11e88c030671ac002d2-586d77860e47b4f5.elb.us-west-2.amazonaws.com
{"message":"Hello"}
```

