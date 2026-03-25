<h2 align='center'> Deploy Web Application via Kubernetes </h2>


<hr>

<h4 align='center'> Hands 0n  </h4>

<hr>

**Step-1:- Run a Pod**
```bash
kubectl run apache-pod --image=httpd
```
![Run Pod](./Images/1.png)



**Step-2:- List Pods**
```bash
kubectl get pods
```
![List Pod](./Images/2.png)



**Step-3:- Inspect Pod**
```bash
kubectl describe pod apache-pod
```
![Inspect Pod](./Images/3.png)


**Step-4:- Access the App**
```bash
kubectl port-forward pod/apache-pod 8081:80
```
![Access Pod](./Images/4.png)


**Step-5:- Verify**
```
http://localhost:8081
```
![Verify](./Images/5.png)

You should see:
→ Apache default page (“It works!”)



**Step-6:- Delete Pod**
```bash
kubectl delete pod apache-pod
```
![Delete Pod](./Images/6.png)

_Insight:_
Same as before:
* Pod disappears permanently
* No self-healing


**Step-7:- Create Deployment**
```bash
kubectl create deployment apache --image=httpd
```
![Create Deployment](./Images/7.png)


**Step-8:- Check**
```bash
kubectl get deployments
kubectl get pods
```
![Check Pod](./Images/8.png)



**Step-9:- Expose Deployment**
```bash
kubectl expose deployment apache --port=80 --type=NodePort
```
![Expose Ports](./Images/9.png)


**Step-10:- Access again**
```bash
kubectl port-forward service/apache 8082:80
```
![Port Forwarding](./Images/10.png)


**Step-11:- Verify**
```
http://localhost:8082
```
![Verify](./Images/11.png)


**Step-12:- Scale Deployment**
```bash
kubectl scale deployment apache --replicas=2
```
![Scale Pod](./Images/12.png)


**Step-13:- List Pods**
```bash
kubectl get pods
```
![Run Pod](./Images/13.png)


**Step-14:- Break the App**
```bash
kubectl set image deployment/apache httpd=wrongimage
```
![Break](./Images/14.png)


**Step-15:- List Pods**
```bash
kubectl get pods
```
![List Pod](./Images/15.png)



**Step-16:- Diagnose**
```bash
kubectl describe pod <pod-name>
```
![Diagnose](./Images/16.png)

_Look for:_
* `ImagePullBackOff`
* error messages



**Step-17:- Fix It**
```bash
kubectl set image deployment/apache httpd=httpd
```
![Fix](./Images/17.png)



**Step-18:- Exec into Pod**
```bash
kubectl exec -it <pod-name> -- /bin/bash
```
![Exec in Pod](./Images/18.png)


**Step-19:- Now inside container:**
```bash
ls /usr/local/apache2/htdocs
```
![Inside Pod](./Images/19.png)

_This is where web files are stored._


**Step-20:- Exit**
```bash
exit
```
![Exit Pod](./Images/20.png)


**Step-21:- Delete One Pod**
```bash
kubectl delete pod <one-pod-name>
```
![Delete Pod](./Images/21.png)


**Step-22:- Watch**
```bash
kubectl get pods -w
```
![Watch Pod](./Images/22.png)



**Step-23:- Cleanup**
```bash
kubectl delete deployment apache
kubectl delete service apache
```
![Run Pod](./Images/23.png)


<hr>

<h4 align='center'> Insights  </h4>

<hr>

**Why there is no detached mode**

`kubectl port-forward` is meant as a **temporary debugging tool**, not a background service.

Kubernetes expects:

* Short-lived usage
* Manual control (start → debug → stop)

For long-running exposure, Kubernetes provides proper resources:

* `Service` (NodePort / ClusterIP)
* `Ingress`



**Running in background using `&`**
```bash
kubectl port-forward pod/apache-pod 8081:80 &
```

This sends the process to the background.



**How to identify the process**

>Method 1: Using jobs (current terminal)
```
jobs
```

_Output example:_
```
[1]+  Running   kubectl port-forward pod/apache-pod 8081:80 &
```


**Method 2: Using `ps`**
```bash
ps aux | grep port-forward
```

_Example output:_
```bash
user   12345  ... kubectl port-forward pod/apache-pod 8081:80
```

> Here, `12345` is the **PID (Process ID)**



**How to stop the process**

**Method 1: Using job number**
```
kill %1
```


**Method 2: Using PID**
```
kill 12345
```

**Method 3: Kill all port-forward processes**
```bash
pkill -f port-forward
```


**Better approach (recommended)**
Instead of `&`, use:

**Option 1: `tmux`**
```bash
tmux new -s pf
kubectl port-forward pod/apache-pod 8081:80
```

_Detach:_
```
Ctrl + b, d
```

This is cleaner and easier to manage.


**Option 2: `nohup`**
```bash
nohup kubectl port-forward pod/apache-pod 8081:80 > pf.log 2>&1 &
```


<hr>

<h4 align='center'> Summary  </h4>

<hr>

* `kubectl port-forward` blocks terminal because it runs a **live network tunnel**
* No detached mode because it is meant for **temporary debugging**
* Use `&`, `jobs`, `ps`, and `kill` to manage background processes
* Prefer `tmux` for better control in DevOps workflows

