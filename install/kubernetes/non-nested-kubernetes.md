In Non-nested mode Kubernetes cluster is provisioned side by side with Openstack cluster with networking provided by same Contrail controller.

![Contrail Non-Nested Solution](/images/non-nested-kubernetes.png)

# __Prerequisites__

1. Installed and running Contrail Openstack cluster on either Bare Metal server (or Virtual Machine).
   This cluster should be based on Contrail 5.0 release.

2. Installed and running Kubernetes cluster on same Bare Metal server (or Virtual Machine) as used in step 1.

3. Label the Kubernetes master node using following command:

```
       kubectl label node <node> node-role.opencontrail.org/controller=true
```

4. It is recommended that the Kubernetes master should not be configured with network plugin,
   as it may not be desirable to install vrouter kernel module on the control node.
   However it is left to the user's preference.

5. Provision additional worker Kubernetes virtual machines (with CentOS 7.5 OS) and join them to the Kubernetes cluster provisioned above.

6. If Contrail container images are stored in private/secure docker registry, a kubernetes secret should be created and referenced during creation of single yaml, with credentials of the private docker registry.

```
   kubectl create secret docker-registry <name> --docker-server=<registry> --docker-username=<username> --docker-password=<password> --docker-email=<email> -n <namespace>

   <name>      - name of the secret
   <registry>  - example: hub.juniper.net/contrail
   <username>  - registry user name
   <password>  - registry passcode
   <email>     - registered email of this registry account
   <namespace> - kubernetes namespace where this secret is to be created. 
                 This should be the namespace where you intend to create Contrail pods.

   ```
 
# __Provision__
Follow these steps to provision Contrail Kubernetes cluster side   

1. Clone contrail-container-build repo in any server of your choice.
```
       git clone https://github.com/Juniper/contrail-container-builder.git
```

2. Populate common.env file (located in the top directory of the cloned contrail-container-builder repo) with info corresponding to your cluster and environment.

For you reference, see a sample common.env file with required bare minimum configurations here:  https://github.com/Juniper/contrail-container-builder/blob/master/kubernetes/sample_config_files/common.env.sample.non_nested_mode

***NOTE 1: If Contrail Config API is not secured by keystone, please ensure AUTH_MODE and KEYSTONE_* variables are not configured/present while populating configuration in common.env**

**NOTE 2:
   If Contrail container images are stored in private/secure docker registry, a kubernetes secret should have be created, as 
   documented in pre-requesites. Populate the variable KUBERNETES_SECRET_CONTRAIL_REPO=< secret-name > with the name of the 
   generated secret,in common.env file.**

3. Generate the yaml file as following:
```
       cd <your contrail-container-build-repo>/kubernetes/manifests

       ./resolve-manifest.sh contrail-non-nested-kubernetes.yaml  > non-nested-contrail.yml
```

4. If any of the macros are not specified in common.env, they will have empty string assignments in the "env" ConfigMap in the generated yaml. Make sure such empty macros are removed from the yaml.
   
5. Copy over the file generated from Step 3 to the master node in your Kubernetes cluster.

6. Create contrail components as pods on the Kubernetes cluster, as follows:

```
       kubectl apply -f non-nested-contrail.yml
```
7. Following Contrail pods should be created on the Kubernetes cluster. Notice that contrail-agent pod is created only on the worker node.
```
       [root@b4s403 manifests]# kubectl get pods --all-namespaces -o wide
       NAMESPACE     NAME                             READY     STATUS    RESTARTS   AGE       IP              NODE
       kube-system   contrail-agent-mxkcq             2/2       Running   0          1m        10.84.24.52     b4s402
       kube-system   contrail-kube-manager-glw5m      1/1       Running   0          1m        10.84.24.53     b4s403
```
