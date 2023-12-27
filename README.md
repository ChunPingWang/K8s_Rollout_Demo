---
title: 'K8s Rollout Demo'
disqus: hackmd
---
K8s Rollout Demo
===

Deployment
---
```gherkin=
kubectl create deploy iisi-demo --image=cpingwang/iisi-demo:0.1.0 --replicas=3 --dry-run=client -o yaml > deploy.yaml
```
> deploy.yaml 內容如下：  
```gherkin=
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: iisi-demo
  name: iisi-demo
spec:
  replicas: 3
  selector:
    matchLabels:
      app: iisi-demo
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: iisi-demo
    spec:
      containers:
      - image: cpingwang/iisi-demo:0.1.0
        name: iisi-demo
        resources: {}
status: {}
```
> 執行  
```gherkin=
kubectl apply -f deploy.yaml
```
>驗證  
> 取得 deployment  
![截圖 2023-12-28 上午1.30.09](https://hackmd.io/_uploads/HJ4Vk1cDa.png)
> 取得 pod  
![截圖 2023-12-28 上午1.51.22](https://hackmd.io/_uploads/ryKXEJqvp.png)

Service (type=LoadBalancer)  
---  
>產出 service yaml檔，type為 Load Balancer  
```gherkin=
kubectl expose deploy/iisi-demo --type=LoadBalancer --port=8080  --dry-run=client -o yaml > svc.yaml
```
> svc.yaml 內容如下：  
```gherkin=
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app: iisi-demo
  name: iisi-demo
spec:
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    app: iisi-demo
  type: LoadBalancer
status:
  loadBalancer: {}
```
 
> 執行  
```gherkin=
kubectl apply -f svc.yaml
```
> 查詢狀態  
```gherkin=
kubectl get svc -A
```
  
![截圖 2023-12-28 上午1.39.36](https://hackmd.io/_uploads/BJTPby5Pp.png)

> 驗證  
```gherkin=
curl localhost:8080 # localhost 可視 external-ip 修改
```
  
![截圖 2023-12-28 上午1.10.52](https://hackmd.io/_uploads/S1Ci5CFPT.png)

Update Deployment  
---  
> 修改 deploy.yaml，指定 --image=cpingwang/iisi-demo:0.2.0  
> 或執行下列指令  
```gherkin=  
kubectl set image deployment/iisi-demo iisi-demo=cpingwang/iisi-demo:0.2.0
```
> 查詢歷史紀錄  
```gherkin=  
kubectl rollout history deployment/iisi-demo
```  
![截圖 2023-12-28 上午1.43.05](https://hackmd.io/_uploads/SJIVzy9P6.png)
> 觀察 pod ，已經與前一版本不一樣  
![截圖 2023-12-28 上午1.53.23](https://hackmd.io/_uploads/S1E2NJ5Pa.png)
> 驗證  
```gherkin=
curl localhost:8080 # localhost 可視 external-ip 修改
```  
> 觀察內容，版本已經被換成 0.2.0  
![截圖 2023-12-28 上午1.55.06](https://hackmd.io/_uploads/ryF-SJ9P6.png)
> 可再次修改版本   
```gherkin=
kubectl set image deployment/iisi-demo iisi-demo=cpingwang/iisi-demo:0.3.0
```
> 查詢版本歷史，多出第三個版本  
```gherkin=
kubectl rollout history deploy iisi-demo
```  
![截圖 2023-12-28 上午1.57.35](https://hackmd.io/_uploads/BJAqBy5PT.png)

> 驗證  
```gherkin=
curl localhost:8080 # localhost 可視 external-ip 修改
```  
![截圖 2023-12-28 上午1.58.31](https://hackmd.io/_uploads/BJrRByqvp.png)
> 查看第一個版本  
```gherkin!
kubectl rollout history deploy/iisi-demo --revision=1
```
![截圖 2023-12-28 上午2.00.15](https://hackmd.io/_uploads/HyTNI1qD6.png)
> 回滾至第一個版本  
```gherkin=
kubectl rollout undo deploy iisi-demo --to-revision=1
```
![截圖 2023-12-28 上午2.05.35](https://hackmd.io/_uploads/SJpdDJcD6.png)

> 驗證  
```gherkin=
curl localhost:8080 # localhost 可視 external-ip 修改
```
> 前面查詢不到的原因為，pod 正在做更換，要避免服務中斷的現象，則需要使用藍綠部署  
![截圖 2023-12-28 上午2.06.18](https://hackmd.io/_uploads/B1djDJcv6.png)

###### tags: `Templates` `Documentation`
