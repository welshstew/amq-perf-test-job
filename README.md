```
oc create -f https://gist.githubusercontent.com/welshstew/3cf81ecfcd3d7a945b31faea85be2c48/raw/ece12a2046104381e607760fbe89685748036f39/perf-test-claim.json
oc create -f - <<EOF
---
apiVersion: batch/v1
kind: Job
metadata:
  name: amq-perf-consumer-1
spec:
  parallelism: 1    
  completions: 1    
  template:         
    spec:
      containers:
      - name: perf-consumer-1
        image: 172.30.1.1:5000/openshift/fis-java-openshift:2.0
        command: ["/bin/bash", "-c", "curl -o /var/tmp/create-consumer.sh -s https://gist.githubusercontent.com/welshstew/3cf81ecfcd3d7a945b31faea85be2c48/raw/ece12a2046104381e607760fbe89685748036f39/create-consumer.sh; source /var/tmp/create-consumer.sh"]
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

oc create -f https://gist.githubusercontent.com/welshstew/3cf81ecfcd3d7a945b31faea85be2c48/raw/ece12a2046104381e607760fbe89685748036f39/perf-test-claim.json

/tmp/src


-Dmaven.repo.local="c:\test\repo"


 mvn activemq-perf:consumer -Dfactory.brokerURL=tcp://broker-amq-tcp:61616 -DsysTest.numClients=1 -Dconsumer.durable=true -Dfactory.userName=admin -Dfactory.password=admin -DsysTest.reportDir=/tmp/reportDir -Dmaven.repo.local="/tmp/artifacts"



MAVEN_ARGS=dependency:go-offline


Copying Maven artifacts from /tmp/src/target to /deployments ...
Running: cp *.jar /deployments
/usr/local/s2i/assemble: line 62: cd: /tmp/src/target: No such file or directory
cp: cannot stat '*.jar': No such file or directory
Aborting due to error code 1 for copying artifacts from /tmp/src/target to /deployments
error: build error: non-zero (13) exit code from registry.access.redhat.com/jboss-fuse-6/fis-java-openshift@sha256:4c701b79aeb3fd87147447c1bf445502ceb15343d171b87f47a70bf9aa530263
[swinches@localhost ~]$ docker images
Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get http://%2Fvar%2Frun%2Fdocker.sock/v1.26/images/json: dial unix /var/run/docker.sock: connect: permission denied


