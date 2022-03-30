# Openshift Pipelines to build a jupyter image and deploy 

### Clone the repository and update the parameters
```
#cd /root/alexei/openshift-pipeline-jupyter-notebook
git clone https://github.com/thinkahead/openshift-pipeline
cd openshift-pipeline
vi pipeline/04-pipelinerun.yaml # Edit the deployment name, image etc
vi openshift/deploymentconfig.yaml # Edit the JUPYTER_NOTEBOOK_PASSWORD default test1234
```

### Run the pipeline and wait for it to complete
The build-and-deploy-run pipeline run will build the base image with the build-image (buildah) and deploy it with oc-apply-deployment task.
```
oc apply -f pipeline/ -n alexei
time tkn pr logs build-and-deploy-run -n alexei -f # 25 minutes

NAME            HOST/PORT                                     PATH   SERVICES        PORT       TERMINATION     WILDCARD
madi-pipeline   madi-pipeline-alexei.apps.ies.pbm.ihost.com          madi-pipeline   8888-tcp   edge/Redirect   None
```
### Get the route
Run the digits.ipynb notebook at the url from the route, password test1234.
```
oc get routes
```
Browse to https://madi-pipeline-alexei.edge-system-health-ocp-cf7808d3396a7c1915bd1818afbfb3c0-0000.upi.containers.appdomain.cloud

### If there is an error, you can delete the pipeline run and redeploy
```
oc delete pr build-and-deploy-run -n alexei
oc apply -f pipeline/04-pipelinerun.yaml -n alexei
```

### Delete the deployed jupyter notebook
```
oc -n alexei delete all,configmap,pvc,serviceaccount,rolebinding,buildconfig --selector app=madi-pipeline
```

### Delete the pipeline and pvc
```
oc delete -f pipeline/ -n alexei
```

### Note
The Kind: ClusterTask did not work for me, so I added the following to alexei namespace
```
oc apply -f https://raw.githubusercontent.com/tektoncd/catalog/main/task/git-clone/0.5/git-clone.yaml
oc apply -f https://raw.githubusercontent.com/tektoncd/catalog/main/task/buildah/0.3/buildah.yaml
# DO NOT DO THIS
oc adm policy add-scc-to-group privileged system:authenticated # change the default SCC for the cluster
# Instead add it to the service account
#oc adm policy add-scc-to-user privileged system:serviceaccount:alexei:default
```
https://github.com/RedHatWorkshops/openshiftv3-ops-workshop/blob/master/managing_scc.md

## Echoer ClusterTask
```
oc apply -f echoer-task.yaml
oc get clustertask echoer
tkn clustertask describe echoer
tkn clustertask start echoer --showlog
tkn tr delete echoer-run-f7646
```
