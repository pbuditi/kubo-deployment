# Troubleshooting

## Table of contents

 - [Unable to create Google Storage Bucket when deploying BOSH](#unable-to-create-google-storage-bucket-when-deploying-bosh)
 - [Service account permission errors during BOSH deployment](#service-account-permission-errors-during-bosh-deployment)
 - [I/O timeout when deploying BOSH](#i-o-timeout-when-deploying-bosh)
 - [Unable to login to CredHub](#unable-to-login-to-credhub)
 - [bosh-cli 401 error for UAA](#bosh-cli-401-error-for-uaa)
 - [Master is not running after the update](#master-is-not-running-after-the-update)
 - [Timeout failures during OSS deployment](#timeout-failures-during-oss-deployment)
 - [K8s deployment fails after BOSH is redeployed](#k8s-deployment-fails-after-bosh-is-redeployed)
 - [Worker failure during deployment of second cluster](#worker-failure-during-deployment-of-second-cluster)
 - [Other connectivity issues](#other-connectivity-issues)



## Unable to create Google Storage Bucket when deploying BOSH

The error below occurs when deploying BOSH on GCP

`CmdError{"type":"Bosh::Clouds::CloudError","message":"Creating stemcell: Creating Google Storage Bucket: Post https://www.googleapis.com/storage/v1/b?alt=json\u0026project=cf-pcf-kubo: Get http://169.254.169.254/computeMetadata/v1/instance/service-accounts/default/token: dial tcp 169.254.169.254:80: i/o timeout","ok_to_retry":false}`

### Solution

Make sure that you have logged in with the newly created service account as described in
[GCP Setup](https://github.com/cloudfoundry-incubator/bosh-google-cpi-release/tree/master/docs/bosh#setup).

## Service account permission errors during BOSH deployment

Various service account permission issues may prevent BOSH deployment

### Solution

Verify that the correct permissions are applied in the [Google Cloud Console](https://console.cloud.google.com/iam-admin/iam). If the permissions are set correctly, but you are still experiencing permission issues, try creating a new account with a different name.

## I/O timeout when deploying BOSH

When deploying BOSH via an `sshuttle` connection, the following error might occur:

```
Command 'deploy' failed:
  Deploying:
    Creating instance 'bosh/0':
      Waiting until instance is ready:
        Sending ping to the agent:
          Performing request to agent endpoint 'https://mbus:294a691d057ede1af4f696aab36c4bc5@10.0.0.4:6868/agent':
            Performing POST request:
              Post https://mbus:294a691d057ede1af4f696aab36c4bc5@10.0.0.4:6868/agent: dial tcp 10.0.0.4:6868: i/o timeout
```

where `10.0.0.4` is the BOSH IP address.

### Solution

Restart the `sshuttle` connection.

## Unable to login to CredHub

A BOSH or kubo deployment may fail with the following error:

```
The provided username and password combination are incorrect. Please validate your input and retry your request.
```

### Solution

This is typically caused by version mismatch between the Credhub CLI and backend. The versions must match exactly.
Please make sure that the installed Credhub CLI matches version in the
[requirements section](../README.md#required-software):

```
$ credhub --version
CLI Version: 0.4.0
Server Version: 0.4.0
```

## bosh-cli 401 error for UAA

Strange error when running bosh-cli

```
bosh-cli -e p-kubo deployments
Using environment 'p-kubo.p.kubo.cf-app.com' as '?'

Finding deployments:
  Performing request GET 'https://p-kubo.p.kubo.cf-app.com:25555/deployments':
    Performing GET request:
      Refreshing token: UAA responded with non-successful status code '401' response '{"error":"invalid_token","error_description":"The token expired, was revoked, or the token ID is incorrect: acc3bc0eeb2b4323995f6d5873d9f52e-r"}'

Exit code 1
```

### Solution

Please make sure that you are logged in with `bosh-cli`. The password can be found in `<KUBO_ENV>/creds.yml`, in the `admin_password` field.


## Master is not running after the update

The following error may be displayed when running `deploy_k8s` script

```
Updating instance master: master/20f5c31f-4329-46a7-ae03-484f0a17f6a3 (0) (canary) (00:01:14)
            Error: 'master/0 (20f5c31f-4329-46a7-ae03-484f0a17f6a3)' is not running after update. Review logs for failed jobs: kubernetes-api, kubernetes-controller-manager, kubernetes-scheduler, kubernetes-api-route-registrar
Error: 'master/0 (20f5c31f-4329-46a7-ae03-484f0a17f6a3)' is not running after update. Review logs for failed jobs: kubernetes-api, kubernetes-controller-manager, kubernetes-scheduler, kubernetes-api-route-registrar
```

### Possible solutions

#### Check the CF credentials

Please check the fields `routing-cf-client-id` in `<KUBO_ENV>/director.yml` and `routing-cf-client-secret` in `<KUBO_ENV>/director-secrets.yml` and ensure that the UAAC credentials that you are using are valid. You can use the [UAAC CLI](https://docs.cloudfoundry.org/adminguide/uaa-user-management.html) to create and manage credentials.

#### Check the accessibility of the routing API URL

Please check that the route to the TCP routing API URL is accessible.  The URL is defined in the field `routing-cf-api-url` in your `<KUBO_ENV>/director.yml`.

## Timeout failures during OSS deployment

kubo [OSS deployment](https://github.com/cloudfoundry-incubator/bosh-google-cpi-release/tree/master/docs/cloudfoundry) may fail due to timeouts.

### Solution

Retry the deployment commands. This can be caused by the default preemptable compilation VMs or by failure to resolve the `xip.io` domain.

If you have access to a domain then you can increase reliability by using it for your Cloud Foundry deployment. If you do not mind the increased cost you can remove the [preemptable](https://github.com/cloudfoundry-incubator/bosh-google-cpi-release/tree/master/src/bosh-google-cpi#bosh-resource-pool-options) property from the compilation workers in the Cloud Foundry manifest.

## K8s deployment fails after BOSH is redeployed

When deploying K8S following a redeployment of BOSH, the following error message may be displayed:

```
$ bin/deploy_k8s <KUBO_ENV> <NAME> public
=====================================
|     BOSH K8S Cluster Deployer     |
=====================================

Fetching info:
  Performing request GET 'https://10.0.7.4:25555/info':
    Performing GET request:
      Retry: Get https://10.0.7.4:25555/info: x509: certificate signed by unknown authority (possibly because of "crypto/rsa: verification error" while trying to verify candidate authority certificate "ca")

Exit code 1
```

### Solution

Reset BOSH alias to use new SSL certificate.

```
$ bin/set_bosh_alias <KUBO_ENV>
```

## Worker failure during deployment of second cluster

```
Updating instance worker: worker/aca2bddf-59b1-4802-a9c5-e09aa09a0efd (0) (canary) (00:03:57)
L Error: Action Failed get_task: Task 75bce522-4fba-4295-7b8c-d63d86f7dcc6 result: 1 of 1 post-start scripts failed. Failed Jobs: kubelet.
```

### Solution

Make sure that port specified as `kubernetes-master-port` is not already in use by another kubo cluster. The value in `director.yml` can be overridden using [var-files](./docs/guides/customized-installation.md#generate-manifest-and-deploy) for different clusters.   

## Other connectivity issues

Various connectivity issues may arise during deployment

### Solution

Please make sure that all the host names used in the configuration are resolving to the correct IP addresses.

## Etcd cluster has multiple leaders ("split brain")

A network partition can cause the etcd cluster to have multiple leaders (otherwise known as a "split brain").
This can result in identical queries to the cluster returning different results. Under most circumstances etcd
can recover once the partition has ended, but there are two situations where it may become stuck in this state:

  1. The network partition occurs during the deployment of etcd. This leads to the two partitions becoming
     completely separate clusters. Neither of the two clusters consider the others as a peer, so the issue
     will not resolve itself once the partition has cleared up.
  2. The node that was isolated (e.g. the single node in a two-one split) reboots before the network partition
     has resolved. The majority cluster will continue to function while considering the isolated node to be an
     unhealthy peer, while the isolated node will begin operating independently. Once the partition has been
     resolved, the isolated node will rejoin the majority cluster and any data written into it during the
     partition will be lost.

### Detection

  1. `ssh` into each of the etcd nodes.
  2. Run `/var/vcap/packages/etcd/etcdctl member list`
  3. Compare the results from each of the nodes. If the cluster is healthy, they will all be the same.
     If the cluster is unhealthy, they will have different leaders and possibly a different list of
     members.

### Solution

In order to resolve the split brain, you must intervene manually. The following steps _only_ outline what is necessary
for the isolated node(s) to rejoin the cluster:

  1. First ensure that the etcd nodes may communicate with one another by `ssh`ing into nodes from the majority cluster
     and ensuring that network traffic can flow to the isolated node:

     ```
     $ curl -L http://<isolated-node-address>:7001/version
     {"etcdserver":"3.1.8","etcdcluster":"3.1.0"}
     ```

  2. At this point `etcd-consistency-checker` will likely be reporting that there are two leaders now:

     ```
     $ less /var/vcap/sys/log/etcd/etcd_consistency_checker.log
     ...
     2017/09/06 21:19:23 more than one leader exists: [http://<majority-leader-address>:4001 http://<isolated-node-address>:4001]
     ...
     ```

  3. Reboot etcd on the isolated node to allow the majority-leader to become the lone leader.
     1. `ssh` into the isolated node
     2. Restart etcd through monit:

        ```
        $ sudo -i
        $ /var/vcap/bosh/bin/monit stop etcd
        $ /var/vcap/bosh/bin/monit start etcd
        ```

        __Note:__ If you would prefer the isolated node became the lone leader, you can run the same commands after logging into
        the other leader (from the majority cluster).

  #### Notes

  When rebooting etcd and allowing the majority-leader to become the lone-leader of the cluster, the data contained there becomes canon.
  Any data in the isolated node will be overwritten. Preventing this requires manual curation/replacement through tools like `etcdctl backup`.

  The Pivotal Cloud Foundry Knowledge Base contains additional documentation on recovering in this scenario and can be found
  [here](https://discuss.pivotal.io/hc/en-us/articles/115004051188-How-to-recover-etcd-when-more-than-one-leader-exists-).
