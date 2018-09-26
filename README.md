# installer-e2e
This repo is my setup/notes for running openshift/installer through ci-operator.
I use this to launch a cluster in AWS via openshift/installer, and to run CI against the cluster.
There are a few minor additions to the templates in this repo from those running in CI; I've 
added a secret to transfer aws credentials and quay pull secret.  

First, get a token and login to the OpenShift CI cluster.
Next: Note the files required for the cluster-secrets-aws secret and create them/change paths
accordingly.  The ssh key-pair is meant to be generated/only used for ci-testing. The private
key is used only when running the full conformance test suite, so that's optional.

Then: 

Create a new project in https://api.ci.openshift.org:
 ```bash
oc new-project your-namespace
```

Create a secret to pass your local files: 
 ```bash
oc create secret -n your-namespace generic cluster-profile \
                    --from-file=secret/.awscred \
                    --from-file=secret/pull-secret \
                    --from-file=secret/ssh-publickey \
                    -o yaml --dry-run | oc -n your-namespace apply -f -
```

Now run the ci-operator command: 
```bash
ci-operator -template templates/cluster-launch-installer-e2e-new.yaml \
               -config /path/to/openshift/release/ci-operator/config/openshift/installer/master.yaml \
               -git-ref=your-gh-username/installer@your-branch \
               -namespace=your-namespace
```


You can then access your project in the CI web console, check the pods.

When running the full test suite, conformance test output will be found via
pod logs for pod/dev container/test.  

You can grab the kubeconfig, copy to your local system, and access the cluster running in AWS.
From the CI web console in your project, go to pod/dev container/setup and in the terminal, 
kubeconfig is at /tmp/shared/cluster/generated/auth/kubeconfig.  Copy that to your local system, then
`export KUBECONFIG=/path/to/copied/kubeconfig`.

Clean up your AWS resources from a cluster!!!
1. `git clone git@github.com:openshift/hive.git`
2. `cd hive; make hiveutil`
3. `bin/hiveutil aws-tag-deprovision --cluster-name your-cluster-name --loglevel debug tectonicClusterID=see-aws-console-for-tag`
