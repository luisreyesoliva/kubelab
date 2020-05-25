# IBM Kuberneteando sobre IBM Cloud Workshop

---
# Kuberneteando 3 - Kubernetes Hands-On
---


# Task 4 : Deploy a simple app (optional)

For this tutorial, we have taken a simple Hello World! Node.js application to deploy on Kubernetes as shown. The following is code from our sample app; use one that you have on hand.

``` javascript
  const app = require('express')()

  app.get('/', (req, res) => {
    res.send("Hello from Appsody!");
  });

  var port = 3300;

  var server = app.listen(port, function () {
    console.log("Server listening on " + port);
  })

  module.exports.app = app;
```

Create and clone a github repo:

```
cd
mkdir deploy
cd deploy
git clone https://github.com/IBM/deploy-app-using-tekton-on-kubernetes deploy-app
cd deploy-app/src
```

Check your IBM Cloud registry:

```bash
ibmcloud cr api
```

Results

```bash
# ibmcloud cr api
                           
Registry API endpoint   https://uk.icr.io/api   

OK
```

Take a note of the registry. In that case **uk.icr.io**

Build and push it to IBM Cloud Container registry.

```
ibmcloud cr build -t uk.icr.io/<namespace>/builtApp:1.0 .
```

Results

```bash
# ibmcloud cr build -t uk.icr.io/ireg/hello:1.0 .
Sending build context to Docker daemon   5.12kB
Step 1/7 : FROM node:alpine
 ---> b01d82bd42de
Step 2/7 : COPY app.js /app/app.js
 ---> Using cache
 ---> 962052c0ed3b
Step 3/7 : COPY package.json /app/package.json
 ---> Using cache
 ---> 2bdbf3d2d97c
Step 4/7 : RUN cd /app && npm install
 ---> Using cache
 ---> dc1810bd90ae
Step 5/7 : ENV WEB_PORT 3300
 ---> Using cache
 ---> 92ff94d39dc1
Step 6/7 : EXPOSE  3300
 ---> Using cache
 ---> c7c9ac470b16
Step 7/7 : CMD ["node", "/app/app.js"]
 ---> Using cache
 ---> d96b8cee6467
Successfully built d96b8cee6467
Successfully tagged uk.icr.io/ireg/hello:1.0
The push refers to repository [uk.icr.io/ireg/hello]
debddb1b4cb1: Pushed 
568966cbd2fb: Pushed 
54e067222fef: Pushed 
85b69ca8f2a0: Pushed 
f9ad34829dbf: Pushed 
de5315d732c2: Pushed 
5216338b40a7: Pushed 
1.0: digest: sha256:884c4a49042459477a5d8a4620d0a6e66a1de490ab6b8795bcf4deaf994e237a size: 1783

OK
```

Verify whether the image is uploaded to the container registry

```bash
ibmcloud cr images
```

```bash
# ibmcloud cr images
uk.icr.io/ireg/hello              1.0      884c4a490424   ireg        4 minutes ago   48 MB    No Issues   
```



Update deploy target in deploy.yaml

```
sed -i '' s#IMAGE#uk.icr.io/<namespace>/hello:1.0# deploy.yaml
```

Results:

```yaml
more deploy.yaml
apiVersion: v1
kind: Service
metadata:
  name: app
  labels:
    app: app
spec:
  type: NodePort
  ports:
    - port: 3300
      name: app
      nodePort: 32426
  selector:
    app: app
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app
  labels:
    app: app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: app
  template:
    metadata:
      labels:
        app: app
    spec:
      containers:
      - name: app
        image: uk.icr.io/ireg/hello:1.0
        ports:
        - containerPort: 3300
```

Run deploy configuration

```bash
kubectl create -f deploy.yaml
```

Results

```bash
# kubectl create -f deploy.yaml
service/app created
deployment.apps/app created
```

Verify output - pod and service should be up and running

```bash
# kubectl get pods
NAME                  READY   STATUS    RESTARTS   AGE
app-d78b485cb-567ql   1/1     Running   0          81s

# kubectl get service
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
app          NodePort    172.21.43.135   <none>        3300:32426/TCP   116s
```

Retrieve the public IP of a Kubernetes cluster from your IBM Cloud dashboard

![image-20200314015332345](images/image-20200314015332345-4147212.png)

In a browser use the following URL: 

```http
http://184.172.229.6:32426
```

Results:

![image-20200314015638938](images/image-20200314015638938-4147399.png)





# Conclusion

You have learnt how to create a Kubernetes cluster and see how to configure all the necessary tools (CLI, connection) to manage a cluster and the kubernetes resources (PODs, Services). And you have deployed  an application in that kubernetes cluster !

# End of the lab

------

# Kuberneteando 3 - Kubernetes Hands-On



---
