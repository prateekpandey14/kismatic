# Certificates CLI

Status: Proposal

Certificates are a requirement for running secure Kubernetes clusters. In a cluster bootstrapped by kismatic, all component endpoints are secured using TLS. Furthermore, certificates are also used for user authentication when there is no other authentication mechanism configured.

In order to facilitate the generation and management of certificates, a new command in the kismatic CLI is proposed.

## Use Cases

* As an operator, I want to grant cluster access to a new user by generating a client certificate
* As an operator, I want to grant cluster access to an external system by generating a client certificate
* As an operator, I want to check the validity of all certificates currently in use on the cluster
* As an operator, I want to redeploy certificates that are nearing their expiration date
* As an operator, I want to redeploy certificates with a new CA without having to reinstall my cluster

# Design

The following CLI commands are proposed for introduction to the `kismatic` binary:

## Certificate Generation
A generic, "swiss-army" style command is proposed for generating certificates:

```
kismatic certificates generate <name> [options]
```

Output: Generated key and certificate is placed in the generated directory. Key's filename is `<name>-key.pem`, and certificate's filename is `<name>.pem`.

Pre-conditions:
* CA is in the generated directory

Options:
* `--common-name`: Override the common name. If blank, use `<name>`.
* `--expiry-days`: Specify the number of days this certificate should be valid. Expiration date will be calculated relative to the machine's clock. Defaults to 365.
* `--subj-alt-names`: Comma-separated list of names that should be included in the certificate's subject alternative names field.
* `--organizations`: Comma-separated list of names that should be included in the certificate's organization field.
* `--overwrite`: Overwrite existing certificate if it already exists in the target directory.

## Certificate Validity Check
A command that can be run against an existing cluster to validate the certificates that are deployed on the cluster. Certificates that are withing the warning window will be flagged. Certificates that have expired will be flagged.

```
kismatic certificates validate [options]
```

Output: TBD. Most likely the execution output from an ansible playbook run to keep orchestration in ansible, but potentially a listing of per-node certificates:
```
# kismatic certificates validate
Node: etcd
Certificate          Expires
Etcd server          09/08/2017 10:00 AM

Node: master01
Certificate          Expires         
API server           09/08/2017 10:00 AM
Scheduler client     EXPIRES SOON (06/08/2017 10:00 AM)
...

Node: worker01
Certificate          Expires
Kubelet client       EXPIRED (09/08/2016 10:00 AM)
```

Pre-conditions:
* SSH access to nodes

Options:
* `--warning-window`: Warn about certificates that will expire within this number of days. Defaults to 45 days if not set.

## Certificate Redeployment
A command that can be used to deploy certificates on existing machines. If the certificates have not been generated, the command will generate the certificates for the cluster described in the plan file (As if an installation was being performed), and deploy them to the nodes.

Services are restarted whenever required.

```
kismatic certificates deploy
```

Output: Execution of ansible play for deploying certs.

Pre-conditions:
* SSH access to nodes

# Other considerations
* Ensure that our certificate validation code allows for custom SANs and organizations. I belive this
is already the case, but we should verify.
* Have to figure out the mechanics of certificate redeployment. I _think_ service accounts 
would have to be recreated if the CA changes, or if the service account signing cert changes.