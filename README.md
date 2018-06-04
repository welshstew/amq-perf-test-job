# amq-perf-test-job

What is it? Overriden s2i scripts that run the `activemq-perf:consumer` and `activemq-perf:producer` tasks from the maven activemq-perf plugin as described at: http://activemq.apache.org/activemq-performance-module-users-manual.html. 

The s2i scripts simply assist with downloading all of the dependencies in the assemble stage.

Once you have built the container, it can then be reused for jobs.


## Building

```
git clone https://github.com/welshstew/amq-perf-test-job.git

#Build the container with the amq-perf-plugin built-in
oc new-build fis-java-openshift:2.0~https://github.com/welshstew/amq-perf-test-job.git

#Create a shared PVC
oc create -f openshift/amq-perf-test-pvc.json
```
Once the build is complete the image should be available in the namespace in which you built it.  In the case on my minishift it is available at: `172.30.1.1:5000/amq/amq-perf-test-job:latest`

## Running a Job

Using the below yaml will create an Openshift Job with multiple containers - one producer, and one consumer, configured against the activemq running inside my amq namespace.  The yaml will have to be reconfigured to your liking.  

Various system variables should be configured, please see the documentation at: http://activemq.apache.org/activemq-performance-module-users-manual.html

```
oc create -f - <<EOF
---
apiVersion: batch/v1
kind: Job
metadata:
  name: amq-perf-test-1
spec:
  parallelism: 1    
  completions: 1    
  template:         
    spec:
      containers:
      - name: perf-producer-1
        image: 172.30.1.1:5000/amq/amq-perf-test-job:latest
        command: ["/bin/bash", "-c", "mvn -f /tmp/src/pom.xml activemq-perf:producer -Dfactory.brokerURL=tcp://broker-amq-tcp:61616 -DsysTest.numClients=1 -Dconsumer.durable=true -Dfactory.userName=admin -Dfactory.password=admin -DsysTest.reportDir=/test/ -DsysTest.reportName=amq-perf-producer-1 -Dmaven.repo.local=/tmp/artifacts/m2"]
        resources:
          limits:
            cpu: 300m
            memory: 512Mi
          requests:
            cpu: 300m
            memory: 512Mi        
        volumeMounts:
        - mountPath: /test
          name: test
      - name: perf-consumer-1
        image: 172.30.1.1:5000/amq/amq-perf-test-job:latest
        command: ["/bin/bash", "-c", "mvn -f /tmp/src/pom.xml activemq-perf:consumer -Dfactory.brokerURL=tcp://broker-amq-tcp:61616 -DsysTest.numClients=1 -Dconsumer.durable=true -Dfactory.userName=admin -Dfactory.password=admin -DsysTest.reportDir=/test/ -DsysTest.reportName=amq-perf-consumer-1 -Dmaven.repo.local=/tmp/artifacts/m2"]
        resources:
          limits:
            cpu: 300m
            memory: 512Mi
          requests:
            cpu: 300m
            memory: 512Mi        
        volumeMounts:
        - mountPath: /test
          name: test          
      volumes:
      - name: test
        persistentVolumeClaim:
          claimName: perf-test-claim
      restartPolicy: Never
...
EOF
```

## Inspecting the results

Check the logs in your containers for the job:

```
oc logs jobs/amq-perf-test-1 -c perf-producer-1
...
########################################
####    SYSTEM CPU USAGE SUMMARY    ####
########################################
Total Blocks Received: 1615560
Ave Blocks Received: 6731.5
Total Blocks Sent: 1822740
Ave Blocks Sent: 7594.75
Total Context Switches: 1830966
Ave Context Switches: 7629.025
Total User Time: 7433
Ave User Time: 30.970833333333335
Total System Time: 5084
Ave System Time: 21.183333333333334
Total Idle Time: 10882
Ave Idle Time: 45.34166666666667
Total Wait Time: 277
Ave Wait Time: 1.1541666666666666
[INFO] Created performance report: /test/amq-perf-producer-1.xml
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
...

```

Also, we had a shared persistent storage setup where the reports are produced:

```
[user@localhost amq-perf-test-job]$ oc get pv | grep perf-test-claim
pv0059    100Gi      RWO,ROX,RWX    Recycle          Bound       amq/perf-test-claim                             5d

#Export the pv so we can see where the volume is:

[user@localhost amq-perf-test-job]$ oc get pv | grep perf-test-claim | awk '{print $1}' | xargs oc export pv
apiVersion: v1
kind: PersistentVolume
metadata:
  annotations:
    pv.kubernetes.io/bound-by-controller: "yes"
  creationTimestamp: null
  labels:
    volume: pv0059
  name: pv0059
spec:
  accessModes:
  - ReadWriteOnce
  - ReadWriteMany
  - ReadOnlyMany
  capacity:
    storage: 100Gi
  claimRef:
    apiVersion: v1
    kind: PersistentVolumeClaim
    name: perf-test-claim
    namespace: amq
    resourceVersion: "239776"
    uid: c7b56267-6801-11e8-bf9f-52540038bbcc
  hostPath:
    path: /var/lib/minishift/openshift.local.pv/pv0059
    type: ""
  persistentVolumeReclaimPolicy: Recycle
status: {}

```
In this instance the volume is in a hostPath because I am running on minishift.  We are able to inspect the xml files here.

```
[user@localhost amq-perf-test-job]$ minishift ssh
Last login: Mon Jun  4 12:21:20 2018 from 192.168.42.1
[docker@minishift ~]$ ll /var/lib/minishift/openshift.local.pv/pv0059
total 144
-rw-r--r--. 1 185 root 70638 Jun  4 12:11 amq-perf-consumer-1.xml
-rw-r--r--. 1 185 root 70649 Jun  4 12:11 amq-perf-producer-1.xml
```


## Configuring to run against a queue

Just supplying a different job config here with different parameters passed to the plugin:

```
oc create -f - <<EOF
---
apiVersion: batch/v1
kind: Job
metadata:
  name: amq-perf-queue-1
spec:
  parallelism: 1    
  completions: 1    
  template:         
    spec:
      containers:
      - name: queue-producer-1
        image: 172.30.1.1:5000/amq/amq-perf-test-job:latest
        command: ["/bin/bash", "-c", "mvn -f /tmp/src/pom.xml activemq-perf:producer -Dfactory.brokerURL=tcp://broker-amq-tcp:61616 -DsysTest.numClients=1 -Dconsumer.durable=true -DsysTest.samplers=tpu -Dproducer.destName=queue://my.queue -Dproducer.deliveryMode=persistent -Dproducer.sendType=count -Dproducer.sendCount=10000 -Dproducer.sessTransacted=true -Dfactory.userName=admin -Dfactory.password=admin -DsysTest.reportDir=/test/ -DsysTest.reportName=amq-queue-producer-1 -Dmaven.repo.local=/tmp/artifacts/m2"]
        resources:
          limits:
            cpu: 300m
            memory: 512Mi
          requests:
            cpu: 300m
            memory: 512Mi        
        volumeMounts:
        - mountPath: /test
          name: test
      - name: queue-consumer-1
        image: 172.30.1.1:5000/amq/amq-perf-test-job:latest
        command: ["/bin/bash", "-c", "mvn -f /tmp/src/pom.xml activemq-perf:consumer -Dfactory.brokerURL=tcp://broker-amq-tcp:61616 -Dconsumer.sessTransacted=true -Dconsumer.destName=queue://my.queue -Dconsumer.recvCount=10000 -DsysTest.numClients=1 -Dconsumer.recvType=count -Dfactory.userName=admin -Dfactory.password=admin -DsysTest.reportDir=/test/ -DsysTest.reportName=amq-queue-consumer-1 -Dmaven.repo.local=/tmp/artifacts/m2"]
        resources:
          limits:
            cpu: 300m
            memory: 512Mi
          requests:
            cpu: 300m
            memory: 512Mi        
        volumeMounts:
        - mountPath: /test
          name: test          
      volumes:
      - name: test
        persistentVolumeClaim:
          claimName: perf-test-claim
      restartPolicy: Never
...
EOF
```