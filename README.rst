=================
Kubernetes on AWS
=================

**WORK IN PROGRESS**

This repo contains some scripts and templates to provision Kubernetes clusters on AWS using Cloud Formation and CoreOS.

**Consider this very early test stuff, many values are hardcoded and we are only trying to solve or own specific Zalando user case!**

It was initially based on `kube-aws`_, but we decided to diverge from it:

* kube-aws uses a single master EC2 instance --- we want to have an ASG for the master nodes (probably running with size 1 usually, but having the option for more, e.g. during updates/migrations etc)
* kube-aws runs etcd on the master --- we want to run etcd separately (currently we use our own 3 node etcd appliance with DNS discovery (SRV records))
* kube-aws does not configure an ELB for the API server --- we want to terminate SSL at ELB in order to leverage existing SSL infrastructure (including ACM)
* kube-aws uses a single CF template --- we want to split into at least 3 CF templates to facilitate future upgrades and extra node pools (one for etcd cluster, one for master and one for worker nodes)

We therefore adapted the generated Cloud Formation to YAML and are using our own `Senza Cloud Formation tool`_ for deployment (it's not doing any magic, but e.g. makes ELB+DNS config easy).

Notes
=====

* Node and user authentication is done via tokens (using the webhook feature)
* SSL client-cert authentication is disabled
* Many values are hardcoded
* Secrets (e.g. shared token) are not KMS-encrypted


Assumptions
===========

* The AWS account has a single Route53 hosted zone including a proper SSL cert (you can use the free ACM service)
* The VPC has at least one public subnet per AZ (either AWS default VPC setup or public subnet named "dmz-<REGION>-<AZ>")
* The VPC is in region eu-central-1 or eu-west-1
* etcd cluster is available via DNS discovery (SRV records) at etcd.<YOUR-HOSTED-ZONE>
* OAuth Token Info is available to validate user tokens


Usage
=====

.. code-block:: bash

    $ sudo pip3 install -U stups-senza awscli # install Senza and AWS CLI
    $ # login to AWS with right region
    $ cd cluster
    $ ./cluster.py create <STACK_NAME> <VERSION> # e.g. ./cluster.py create kube-aws-test 1

This will bootstrap a new cluster and make the API server available as https://<STACK_NAME>-<VERSION>.<YOUR-HOSTED-ZONE-DOMAIN>.

The authorization webhook will require the user to have the group "<ACCOUNT_ALIAS_WITHOUT_PREFIX>-<STACK_NAME>", i.e. if the AWS account alias is "myorg-myteam" and the stack name is "kube-aws-test" then the required group is "myteam-kube-aws-test".


.. _kube-aws: https://github.com/coreos/coreos-kubernetes/tree/master/multi-node/aws
.. _Senza Cloud Formation tool: https://github.com/zalando-stups/senza
