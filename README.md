# amq-perf-test-job

What is it? Overriden s2i scripts that run the `activemq-perf:consumer` task from the maven plugin in order to build the container (download all of the dependencies locally) - mainly because I couldn't figure out how `mvn dependency:go-offline` works.  And this was workds anyway.

## Building

```
oc new-build fis-java-openshift:2.0~https://github.com/welshstew/amq-perf-test-job.git
```

## Running Jobs

```
oc create -f openshift/amq-perf-test-pvc.json
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
        command: ["/bin/bash", "-c", "/tmp/src/mvn activemq-perf:consumer -Dfactory.brokerURL=tcp://broker-amq-tcp:61616 -DsysTest.numClients=1 -Dconsumer.durable=true -Dfactory.userName=admin -Dfactory.password=admin -DsysTest.reportDir=/test -Dmaven.repo.local=/tmp/artifacts/m2"]
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


 mvn activemq-perf:consumer -Dfactory.brokerURL=tcp://broker-amq-tcp:61616 -DsysTest.numClients=1 -Dconsumer.durable=true -Dfactory.userName=admin -Dfactory.password=admin -DsysTest.reportDir=/tmp -Dmaven.repo.local="/tmp/artifacts/m2"



mvn activemq-perf:consumer -Dfactory.brokerURL=tcp://broker-amq-tcp:61616 -DsysTest.numClients=1 -Dconsumer.durable=true -Dfactory.userName=admin -Dfactory.password=admin -DsysTest.reportDir=/tmp/reportDir -Dmaven.repo.local=/home/jboss/.m2/





MAVEN_ARGS=dependency:go-offline

oc new-build fis-java-openshift:2.0~https://github.com/welshstew/activemq-perftest.git


    strategy:
      sourceStrategy:
        env:
        - name: KIE_CONTAINER_DEPLOYMENT
          value: processserver-library=org.openshift.quickstarts:processserver-library:1.4.0.Final
        - name: MAVEN_MIRROR_URL
          value: http://192.168.122.50:8081/nexus/content/groups/public/
        - name: ARTIFACT_DIR
        forcePull: true
        from:
          kind: ImageStreamTag
          name: jboss-processserver64-openshift:1.3
          namespace: openshift
      type: Source
    successfulBuildsHistoryLimit: 5








Copying Maven artifacts from /tmp/src/target to /deployments ...
Running: cp *.jar /deployments
/usr/local/s2i/assemble: line 62: cd: /tmp/src/target: No such file or directory
cp: cannot stat '*.jar': No such file or directory
Aborting due to error code 1 for copying artifacts from /tmp/src/target to /deployments
error: build error: non-zero (13) exit code from registry.access.redhat.com/jboss-fuse-6/fis-java-openshift@sha256:4c701b79aeb3fd87147447c1bf445502ceb15343d171b87f47a70bf9aa530263


oc new-app fis-java-openshift:2.0~https://github.com/welshstew/amq-perf-test-job.git
