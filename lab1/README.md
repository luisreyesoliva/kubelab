# IBM Kuberneteando sobre IBM Cloud Workshop

---
# Kuberneteando 3 - Kubernetes Hands-On
---

# Task 1 : Deploy a simple node app (optional)

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

Check your IBM Cloud registry:

```bash
ibmcloud cr api
```

Results

```bash
# ibmcloud cr api
                                       
Punto final de la API de Registry   https://us.icr.io/api   

OK
```

Take a note of the registry. In that case **us.icr.io**

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




# Lab 1. Deploy and scale your first application

Learn how to deploy an application to a Kubernetes cluster hosted within
the IBM Container Service.

# 1. Deploy your application

In this part of the lab we will deploy an application called `guestbook`
that has already been built and uploaded to DockerHub under the name
`ibmcom/guestbook:v1`.

1. Start by running `guestbook`:

   ```$ kubectl run guestbook --image=ibmcom/guestbook:v1```

   This action will take a bit of time. To check the status of the running application,
   you can use `$ kubectl get pods`.

   You should see output similar to the following:

   ```console
   $ kubectl get pods
   NAME                          READY     STATUS              RESTARTS   AGE
   guestbook-59bd679fdc-bxdg7    0/1       ContainerCreating   0          1m
   ```
   Eventually, the status should show up as `Running`.
   
   ```console
   $ kubectl get pods
   NAME                          READY     STATUS              RESTARTS   AGE
   guestbook-59bd679fdc-bxdg7    1/1       Running             0          1m
   ```
   
   The end result of the run command is not just the pod containing our application containers,
   but a Deployment resource that manages the lifecycle of those pods.
 
   
3. Once the status reads `Running`, we need to expose that deployment as a
   service so we can access it through the IP of the worker nodes.
   The `guestbook` application listens on port 3000.  Run:

   ```console
   $ kubectl expose deployment guestbook --type="NodePort" --port=3000
   service "guestbook" exposed
   ```

4. To find the port used on that worker node, examine your new service:

   ```console
   $ kubectl get service guestbook
   NAME        TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
   guestbook   NodePort   10.10.10.253   <none>        3000:31208/TCP   1m
   ```
   
   We can see that our `<nodeport>` is `31208`. We can see in the output the port mapping from 3000 inside 
   the pod exposed to the cluster on port 31208. This port in the 31000 range is automatically chosen, 
   and could be different for you.

5. `guestbook` is now running on your cluster, and exposed to the internet. We need to find out where it is accessible.
   The worker nodes running in the container service get external IP addresses.
   Run `$ ibmcloud cs workers <name-of-cluster>`, and note the public IP listed on the `<public-IP>` line.
   
   ```console
   $ ibmcloud cs workers osscluster
   OK
   ID                                                 Public IP        Private IP     Machine Type   State    Status   Zone    Version  
   kube-hou02-pa1e3ee39f549640aebea69a444f51fe55-w1   173.193.99.136   10.76.194.30   free           normal   Ready    hou02   1.5.6_1500*
   ```
   
   We can see that our `<public-IP>` is `173.193.99.136`.
   
6. Now that you have both the address and the port, you can now access the application in the web browser
   at `<public-IP>:<nodeport>`. In the example case this is `173.193.99.136:31208`.
   
Congratulations, you've now deployed an application to Kubernetes!

    
# 2. Scale apps with replicas

A *replica* is a copy of a pod that contains a running service. By having
multiple replicas of a pod, you can ensure your deployment has the available
resources to handle increasing load on your application.

1. `kubectl` provides a `scale` subcommand to change the size of an
   existing deployment. Let's increase our capacity from a single running instance of
   `guestbook` up to 10 instances:

   ``` console
   $ kubectl scale --replicas=10 deployment guestbook
   deployment "guestbook" scaled
   ```

   Kubernetes will now try to make reality match the desired state of
   10 replicas by starting 9 new pods with the same configuration as
   the first.

4. To see your changes being rolled out, you can run:
   `kubectl rollout status deployment guestbook`.

   The rollout might occur so quickly that the following messages might
   _not_ display:

   ```console
   $ kubectl rollout status deployment guestbook
   Waiting for rollout to finish: 1 of 10 updated replicas are available...
   Waiting for rollout to finish: 2 of 10 updated replicas are available...
   Waiting for rollout to finish: 3 of 10 updated replicas are available...
   Waiting for rollout to finish: 4 of 10 updated replicas are available...
   Waiting for rollout to finish: 5 of 10 updated replicas are available...
   Waiting for rollout to finish: 6 of 10 updated replicas are available...
   Waiting for rollout to finish: 7 of 10 updated replicas are available...
   Waiting for rollout to finish: 8 of 10 updated replicas are available...
   Waiting for rollout to finish: 9 of 10 updated replicas are available...
   deployment "guestbook" successfully rolled out
   ```

5. Once the rollout has finished, ensure your pods are running by using:
   `kubectl get pods`.

   You should see output listing 10 replicas of your deployment:

   ```console
   $ kubectl get pods
   NAME                        READY     STATUS    RESTARTS   AGE
   guestbook-562211614-1tqm7   1/1       Running   0          1d
   guestbook-562211614-1zqn4   1/1       Running   0          2m
   guestbook-562211614-5htdz   1/1       Running   0          2m
   guestbook-562211614-6h04h   1/1       Running   0          2m
   guestbook-562211614-ds9hb   1/1       Running   0          2m
   guestbook-562211614-nb5qp   1/1       Running   0          2m
   guestbook-562211614-vtfp2   1/1       Running   0          2m
   guestbook-562211614-vz5qw   1/1       Running   0          2m
   guestbook-562211614-zksw3   1/1       Running   0          2m
   guestbook-562211614-zsp0j   1/1       Running   0          2m
   ```

**Tip:** Another way to improve availability is to
[add clusters and regions](https://console.bluemix.net/docs/containers/cs_planning.html#cs_planning_cluster_config)
to your deployment, as shown in the following diagram:

![HA with more clusters and regions](../images/cluster_ha_roadmap.png)

# 2. Update and roll back apps

Kubernetes allows you to do rolling upgrade of your application to a new
container image. This allows you to easily update the running image and also allows you to
easily undo a rollout if a problem is discovered during or after deployment.

In the previous lab, we used an image with a `v1` tag. For our upgrade
we'll use the image with the `v2` tag.

To update and roll back:
1. Using `kubectl`, you can now update your deployment to use the
   `v2` image. `kubectl` allows you to change details about existing
   resources with the `set` subcommand. We can use it to change the
   image being used.

    ```$ kubectl set image deployment/guestbook guestbook=ibmcom/guestbook:v2```

   Note that a pod could have multiple containers, each with its own name.
   Each image can be changed individually or all at once by referring to the name.
   In the case of our `guestbook` Deployment, the container name is also `guestbook`.
   Multiple containers can be updated at the same time.
   ([More information](https://kubernetes.io/docs/user-guide/kubectl/kubectl_set_image/).)

3. Run `kubectl rollout status deployment/guestbook` to check the status of
   the rollout. The rollout might occur so quickly that the following messages
   might _not_ display:

   ```console
   $ kubectl rollout status deployment/guestbook
   Waiting for rollout to finish: 2 out of 10 new replicas have been updated...
   Waiting for rollout to finish: 3 out of 10 new replicas have been updated...
   Waiting for rollout to finish: 3 out of 10 new replicas have been updated...
   Waiting for rollout to finish: 3 out of 10 new replicas have been updated...
   Waiting for rollout to finish: 4 out of 10 new replicas have been updated...
   Waiting for rollout to finish: 4 out of 10 new replicas have been updated...
   Waiting for rollout to finish: 4 out of 10 new replicas have been updated...
   Waiting for rollout to finish: 4 out of 10 new replicas have been updated...
   Waiting for rollout to finish: 4 out of 10 new replicas have been updated...
   Waiting for rollout to finish: 5 out of 10 new replicas have been updated...
   Waiting for rollout to finish: 5 out of 10 new replicas have been updated...
   Waiting for rollout to finish: 5 out of 10 new replicas have been updated...
   Waiting for rollout to finish: 6 out of 10 new replicas have been updated...
   Waiting for rollout to finish: 6 out of 10 new replicas have been updated...
   Waiting for rollout to finish: 6 out of 10 new replicas have been updated...
   Waiting for rollout to finish: 7 out of 10 new replicas have been updated...
   Waiting for rollout to finish: 7 out of 10 new replicas have been updated...
   Waiting for rollout to finish: 7 out of 10 new replicas have been updated...
   Waiting for rollout to finish: 7 out of 10 new replicas have been updated...
   Waiting for rollout to finish: 8 out of 10 new replicas have been updated...
   Waiting for rollout to finish: 8 out of 10 new replicas have been updated...
   Waiting for rollout to finish: 8 out of 10 new replicas have been updated...
   Waiting for rollout to finish: 8 out of 10 new replicas have been updated...
   Waiting for rollout to finish: 9 out of 10 new replicas have been updated...
   Waiting for rollout to finish: 9 out of 10 new replicas have been updated...
   Waiting for rollout to finish: 9 out of 10 new replicas have been updated...
   Waiting for rollout to finish: 1 old replicas are pending termination...
   Waiting for rollout to finish: 1 old replicas are pending termination...
   Waiting for rollout to finish: 1 old replicas are pending termination...
   Waiting for rollout to finish: 9 of 10 updated replicas are available...
   Waiting for rollout to finish: 9 of 10 updated replicas are available...
   Waiting for rollout to finish: 9 of 10 updated replicas are available...
   deployment "guestbook" successfully rolled out
   ```

4. Test the application as before, by accessing `<public-IP>:<nodeport>` 
   in the browser to confirm your new code is active.

   Remember, to get the "nodeport" and "public-ip" use:

   `$ kubectl describe service guestbook`
   and
   `$ ibmcloud cs workers <name-of-cluster>`

   To verify that you're running "v2" of guestbook, look at the title of the page,
   it should now be `Guestbook - v2`

5. If you want to undo your latest rollout, use:
   ```console
   $ kubectl rollout undo deployment guestbook
   deployment "guestbook"
   ```

   You can then use `kubectl rollout status deployment/guestbook` to see
   the status.
   
6. When doing a rollout, you see references to *old* replicas and *new* replicas.
   The *old* replicas are the original 10 pods deployed when we scaled the application.
   The *new* replicas come from the newly created pods with the different image.
   All of these pods are owned by the Deployment.
   The deployment manages these two sets of pods with a resource called a ReplicaSet.
   We can see the guestbook ReplicaSets with:
   ```console
   $ kubectl get replicasets -l run=guestbook
   NAME                   DESIRED   CURRENT   READY     AGE
   guestbook-5f5548d4f    10        10        10        21m
   guestbook-768cc55c78   0         0         0         3h
   ```

Before we continue, let's delete the application so we can learn about
a different way to achieve the same results:

 To remove the deployment, use `kubectl delete deployment guestbook`.

 To remove the service, use `kubectl delete service guestbook`.

Congratulations! You deployed the second version of the app. Lab 1
is now complete.


# End of the lab

------

# Kuberneteando 3 - Kubernetes Hands-On



---
