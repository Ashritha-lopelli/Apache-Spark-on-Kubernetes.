<h1>Create a cluster on GKE</h1>

```
gcloud container clusters create spark --num-nodes=1 --machine-type=e2-highmem-2 --region=us-central1
```

<h1>Create image and deploy spark to kubernetes</h1>

```
helm repo add stable https://charts.helm.sh/stable
helm repo update
helm install nfs stable/nfs-server-provisioner \
--set persistence.enabled=true,persistence.size=5Gi
```
<h2>create a persistent disk volume and a pod to use NFS - create a yaml file with name spar-pvc.yaml and insert the code
</h2>

```
vi spark-pvc.yaml
```
<h2>Apply the above yaml descriptor</h2>

```
kubectl apply -f spark-pvc.yaml
```
<h2>Create and prepare your application JAR file</h2>

```
docker run -v /tmp:/tmp -it bitnami/spark -- find /opt/bitnami/spark/examples/jars/ -name spark-examples* -exec cp {} /tmp/my.jar \;
```

<h2>Add a test file with a line of words that we will be using later for the word count test</h2>

```
echo "the quick brown fox the fox ate the mouse how now brown cow" > /tmp/test.txt
```

<h2>Copy the JAR file containing the application, and any other required files, to the PVC using the mount point.</h2>

```
kubectl cp /tmp/my.jar spark-data-pod:/data/my.jar	
kubectl cp /tmp/test.txt spark-data-pod:/data/test.txt
```

<h2>Make sure the files a inside the persistent volume.</h2>

```
kubectl exec -it spark-data-pod -- ls -al /data
```

<h2>Deploy Apache Spark on Kubernetes using the shared volume spark-chart.yaml</h2>

```
nano spark-chart.yaml
cat spark-chart.yaml
```

<h2>Check the pods</h2>

```
kubectl get pods
```

<h2>11.	Deploy Apache Spark on the Kubernetes cluster using the Bitnami</h2>

```
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install spark bitnami/spark -f spark-chart.yaml
```

<h2>Get the external IP of the running pod</h2>

```
kubectl get svc -l "app.kubernetes.io/instance=spark,app.kubernetes.io/name=spark"
```

<h3>Open the external ip on your browser</h3>

<h1>Word Count on Spark</h1>

```
kubectl run --namespace default spark-client --rm --tty -i --restart='Never' \
    --image docker.io/bitnami/spark:3.4.1-debian-11-r3 \
    -- spark-submit --master spark://34.27.61.122:7077 \
    --deploy-mode cluster \
    --class org.apache.spark.examples.JavaWordCount \
    /data/my.jar /data/test.txt
```
<h3>On the browser, see this task finished </h3>

<h1>View the output of the completed jobs</h1>

<h2>Get the name of the worker node</h2>

```
kubectl get pods -o wide | grep WORKER-NODE-ADDRESS
```
<h2>Execute this pod and see the result of the finished tasks</h2>

```
kubectl exec -it <Worker node name> -- bash
cd /opt/bitnami/spark/work 
cat <task-name>/stdout
```
The task name here is the Submission ID in the completed Drivers section of the URL

<h1>Running python PageRank onPySpark on the pods</h1>

<h2>Execute the spark master pods  and Go to the directory where pagerank.py located</h2>

```
kubectl exec -it spark-master-0 – bash
cd /opt/bitnami/spark/examples/src/main/python
```

<h2>Run the pagerank using pyspark</h2>

Note that, /opt is an example directory, you can enter any directory you like, and 2 is the number of iterations you want the pagerank to run, you can also change to any numbers you like

```
spark-submit pagerank.py /opt 2
```












