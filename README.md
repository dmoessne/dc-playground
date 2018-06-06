# dc-playground
playin around with deployment configs

* We start with an empty project and the following very simple Template `kubia-dc.yaml`

<pre>
# oc project
Using project "test01" on server "https://openshift-cluster.example.com:8443".
# 
# oc get all
No resources found.
# 
# cat kubia-dc.yaml 
apiVersion: v1
kind: DeploymentConfig
metadata:
  name: kubia-dc
  labels:
    app: kubia
spec:
  replicas: 2
  matchLabels:
    app: kubia
  template:
    metadata:
      labels:
        app: kubia
    spec:
      containers:
      - name: kubia
        image: dmoessne/kubia:7.0
# 
</pre>

 * We create it and watch the pods getting created

<pre>
# oc create -f kubia-dc.yaml
deploymentconfig "kubia-dc" created
# 
# 
# oc get all
NAME                         REVISION   DESIRED   CURRENT   TRIGGERED BY
deploymentconfigs/kubia-dc   1          2         2         config

NAME                   READY     STATUS              RESTARTS   AGE
po/kubia-dc-1-deploy   1/1       Running             0          11s
po/kubia-dc-1-jbtld    0/1       ContainerCreating   0          2s
po/kubia-dc-1-lvcgl    0/1       ContainerCreating   0          2s

NAME            DESIRED   CURRENT   READY     AGE
rc/kubia-dc-1   2         2         0         11s
# 
# oc get po 
NAME               READY     STATUS    RESTARTS   AGE
kubia-dc-1-jbtld   1/1       Running   0          57s
kubia-dc-1-lvcgl   1/1       Running   0          57s
# 
</pre>

* After everything is running, check containers running with image `dmoessne/kubia:7.0`

<pre>
# oc describe po/kubia-dc-1-jbtld |grep Image
    Image:          dmoessne/kubia:7.0
    Image ID:       docker-pullable://docker.io/dmoessne/kubia@sha256:46867b473723e6d6a34ca3d0387d0048d4ce93b1820f88f3c02263f424a308c4
# 
# oc describe po/kubia-dc-1-lvcgl |grep Image
    Image:          dmoessne/kubia:7.0
    Image ID:       docker-pullable://docker.io/dmoessne/kubia@sha256:46867b473723e6d6a34ca3d0387d0048d4ce93b1820f88f3c02263f424a308c4
# 
# oc get deploymentconfigs/kubia-dc -o yaml |grep image
      - image: dmoessne/kubia:7.0
        imagePullPolicy: IfNotPresent
# 
</pre>

* Edit deploymentconfig to use a newer image and watch that pods are getting recreated using it 

<pre>
# oc edit deploymentconfigs/kubia-dc
deploymentconfig "kubia-dc" edited
#
# oc get deploymentconfigs/kubia-dc -o yaml |grep image
      - image: dmoessne/kubia:7.1
        imagePullPolicy: IfNotPresent
# 
# oc get all
NAME                         REVISION   DESIRED   CURRENT   TRIGGERED BY
deploymentconfigs/kubia-dc   2          2         2         config

NAME                  READY     STATUS        RESTARTS   AGE
po/kubia-dc-1-jbtld   1/1       Terminating   0          4m
po/kubia-dc-1-lvcgl   1/1       Terminating   0          4m
po/kubia-dc-2-h8dcd   1/1       Running       0          20s
po/kubia-dc-2-nj4gh   1/1       Running       0          27s

NAME            DESIRED   CURRENT   READY     AGE
rc/kubia-dc-1   0         0         0         5m
rc/kubia-dc-2   2         2         2         37s
# 
#  oc get deploymentconfigs/kubia-dc -o yaml |grep image
      - image: dmoessne/kubia:7.1
        imagePullPolicy: IfNotPresent
# 
# 
# 
# oc describe po/kubia-dc-2-h8dcd |grep Image
    Image:          dmoessne/kubia:7.1
    Image ID:       docker-pullable://docker.io/dmoessne/kubia@sha256:b2c822495b4b0f4592fcc69abd8a3a18b11d38c90cff0185196f102b68c9a691
# 
# oc describe po/kubia-dc-2-nj4gh |grep Image
    Image:          dmoessne/kubia:7.1
    Image ID:       docker-pullable://docker.io/dmoessne/kubia@sha256:b2c822495b4b0f4592fcc69abd8a3a18b11d38c90cff0185196f102b68c9a691
# 
</pre>

* We'll do that again with an even newer image

<pre>
# oc edit deploymentconfigs/kubia-dc
deploymentconfig "kubia-dc" edited
# 
#  oc get deploymentconfigs/kubia-dc -o yaml |grep image
      - image: dmoessne/kubia:7.2
        imagePullPolicy: IfNotPresent
# 
# oc get po 
NAME               READY     STATUS        RESTARTS   AGE
kubia-dc-2-h8dcd   1/1       Terminating   0          3m
kubia-dc-2-nj4gh   1/1       Terminating   0          3m
kubia-dc-3-rns54   1/1       Running       0          24s
kubia-dc-3-t52sf   1/1       Running       0          18s
# 
# oc describe po/kubia-dc-3-rns54 |grep Image
    Image:          dmoessne/kubia:7.2
    Image ID:       docker-pullable://docker.io/dmoessne/kubia@sha256:03eb3be236b2bf23644f3e2c2b9ae5ed85626216e88033d29f446f632e786da7
# 
# oc describe po/kubia-dc-3-t52sf |grep Image
    Image:          dmoessne/kubia:7.2
    Image ID:       docker-pullable://docker.io/dmoessne/kubia@sha256:03eb3be236b2bf23644f3e2c2b9ae5ed85626216e88033d29f446f632e786da7
# 
# 
</pre>

* we can have a look at previous and current deploymentconfigs using `oc rollout history deploymentconfigs/kubia-dc --revision=<nu>`

<pre>
# 
# oc rollout history deploymentconfigs/kubia-dc --revision=1
deploymentconfigs "kubia-dc" with revision #1
Pod Template:
  Labels:	app=kubia
	deployment=kubia-dc-1
	deploymentconfig=kubia-dc
  Annotations:	openshift.io/deployment-config.latest-version=1
	openshift.io/deployment-config.name=kubia-dc
	openshift.io/deployment.name=kubia-dc-1
  Containers:
   kubia:
    Image:	dmoessne/kubia:7.0
    Port:	<none>
    Environment:	<none>
    Mounts:	<none>
  Volumes:	<none>
# 
# oc rollout history deploymentconfigs/kubia-dc --revision=2
deploymentconfigs "kubia-dc" with revision #2
Pod Template:
  Labels:	app=kubia
	deployment=kubia-dc-2
	deploymentconfig=kubia-dc
  Annotations:	openshift.io/deployment-config.latest-version=2
	openshift.io/deployment-config.name=kubia-dc
	openshift.io/deployment.name=kubia-dc-2
  Containers:
   kubia:
    Image:	dmoessne/kubia:7.1
    Port:	<none>
    Environment:	<none>
    Mounts:	<none>
  Volumes:	<none>
# 
# oc rollout history deploymentconfigs/kubia-dc --revision=3
deploymentconfigs "kubia-dc" with revision #3
Pod Template:
  Labels:	app=kubia
	deployment=kubia-dc-3
	deploymentconfig=kubia-dc
  Annotations:	openshift.io/deployment-config.latest-version=3
	openshift.io/deployment-config.name=kubia-dc
	openshift.io/deployment.name=kubia-dc-3
  Containers:
   kubia:
    Image:	dmoessne/kubia:7.2
    Port:	<none>
    Environment:	<none>
    Mounts:	<none>
  Volumes:	<none>
</pre>

* Before rolling back with `oc rollout undo deploymentconfigs/kubia-dc --to-revision=<nu>` we can check with `--dry-run` what would change if we rolled back 

<pre>
# 
# oc rollout undo deploymentconfigs/kubia-dc --to-revision=1 --dry-run
deploymentconfig "kubia-dc" 
Pod Template:
  Labels:	app=kubia
  Containers:
   kubia:
    Image:	dmoessne/kubia:7.0
    Port:	<none>
    Environment:	<none>
    Mounts:	<none>
  Volumes:	<none>
# 
# oc rollout undo deploymentconfigs/kubia-dc --to-revision=2 --dry-run
deploymentconfig "kubia-dc" 
Pod Template:
  Labels:	app=kubia
  Containers:
   kubia:
    Image:	dmoessne/kubia:7.1
    Port:	<none>
    Environment:	<none>
    Mounts:	<none>
  Volumes:	<none>
</pre>

* Mind the man page: `man oc-rollout-undo`
<pre>
       Revert an application back to a previous deployment
       When  you  run  this command your deployment configuration will be updated to match a previous deployment. By default only
       the pod and container configuration will be changed and scaling or trigger settings will be left as- is. Note that 
       environment variables and volumes are included in rollbacks, so if you've recently updated security credentials in your
       environment your previous deployment may not have the correct values.

       Any  image  triggers  present in the rolled back configuration will be disabled with a warning. This is to help prevent
       your rolled back deployment from being replaced by a trig‐gered deployment soon after your rollback. To re-enable the
       triggers, use the 'oc set triggers --auto' command.

       If you would like to review the outcome of the rollback, pass '--dry-run' to print a human-readable representation of the
       updated deployment configuration  instead  of  executingthe rollback. This is useful if you're not quite sure what the
       outcome will be.
</pre>

* So let's rollback to version one and check image version 

<pre>
# 
# oc rollout undo deploymentconfigs/kubia-dc --to-revision=1 
deploymentconfig "kubia-dc" rolled back
# 
# oc get all
NAME                         REVISION   DESIRED   CURRENT   TRIGGERED BY
deploymentconfigs/kubia-dc   4          2         2         config

NAME                   READY     STATUS              RESTARTS   AGE
po/kubia-dc-3-rns54    1/1       Running             0          3m
po/kubia-dc-3-t52sf    1/1       Terminating         0          3m
po/kubia-dc-4-8dc55    0/1       ContainerCreating   0          5s
po/kubia-dc-4-deploy   1/1       Running             0          20s
po/kubia-dc-4-rkn2j    1/1       Running             0          12s

NAME            DESIRED   CURRENT   READY     AGE
rc/kubia-dc-1   0         0         0         11m
rc/kubia-dc-2   0         0         0         7m
rc/kubia-dc-3   1         1         1         4m
rc/kubia-dc-4   2         2         1         20s
# 
# oc get po  
NAME               READY     STATUS    RESTARTS   AGE
kubia-dc-4-8dc55   1/1       Running   0          49s
kubia-dc-4-rkn2j   1/1       Running   0          56s
# 
# 
# oc describe po/kubia-dc-4-8dc55 |grep Image
    Image:          dmoessne/kubia:7.0
    Image ID:       docker-pullable://docker.io/dmoessne/kubia@sha256:46867b473723e6d6a34ca3d0387d0048d4ce93b1820f88f3c02263f424a308c4
# 
# oc describe po/kubia-dc-4-rkn2j |grep Image
    Image:          dmoessne/kubia:7.0
    Image ID:       docker-pullable://docker.io/dmoessne/kubia@sha256:46867b473723e6d6a34ca3d0387d0048d4ce93b1820f88f3c02263f424a308c4
# 
# 
</pre>

* We're back on the `7.0` image
