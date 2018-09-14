# OpenShift Azure

This Ansible playbook is used to install OpenShift on Microsoft Azure.  Out of the box this playbook will create a three node OpenShift cluster with a Master and two OpenShift nodes on virtual machines in the Azure cloud.  This playbook can be modified to increase the number of nodes in the cluster if so desired.

After checking out this repository, search for the string "INSERT_VALUES_HERE" through out the files and replace that string with the appropriate values for the field.  The files and fields that need this change are listed below.

# Dependencies

- [Ansible](http://docs.ansible.com/ansible/intro_installation.html) (v2.2.1.0)

### Microsoft Azure

- Account with active subscription required

# Required

### group_vars/all.yml
Copy group_vars/all_example to group_vars/all.yml, and the following variables must be set before usage.  Variables that need to be set will be have the placeholder value of ```"$INSERT_VALUE_HERE"```

* The first two values are the DNS domain name (often `redhatgov.io`) and the cluster name, which will form the DNS subdomain:

##### Example:
```
vault_default_domain: "redhatgov.io"
vault_cluster_name:   "nyc-workshop"
```

* Insert your [AWS access and secret access keys](https://docs.aws.amazon.com/general/latest/gr/managing-aws-access-keys.html).

##### Example:
```
vault_aws_access_key_id:     "ABCD3FGIELSKDJD"
vault_aws_secret_access_key: "dlsi#DJEHHSsldkEESJla;89374o30"
```

* The next value can be obtained from the AWS Route53 GUI, or by using the command below the example:

##### Example:
```
vault_aws_route53_zone_id: "Z3TTUI56NVLDJ"
```
OR, use the following command to find the `aws_route53_zone_id`:

<pre>
aws route53 list-hosted-zones --query 'HostedZones[*]' --output text | \
grep '\/hostedzone\/.*<b>${INSERT_VALUE_HERE}</b>' | sed -e 's/.*\///' -e 's/[^a-zA-Z0-9].*//'
</pre>


* Set values so that the OpenShift instances can register with Red Hat Subscription Manager.  You can either use an `activation key / organization id` combination or you can use a `username / password / pool_id` combination.  These two methods are mutually exclusive so *do not define both*.

Set an activation key (which needs to be created in RHSM), and it's accompanying Org ID:

##### Example:
```
rhel_rhsm_activationkey: "my_act_key"
rhel_rhsm_org_id: "12345678"
```
Alternatively, you can use a `username`, `password`, and `pool_id` combination to register with RHSM.

##### Example:
```
rhsm_username:                 "joeuser"
rhsm_password:                 "mypassword"
rhsm_pool_id:                  "393948383948393839293839384"
```

* Define what you want the OpenShift admin user and password to be:

##### Example:
```
vault_openshift_cluster_admin_username: "ocpadmin"
vault_openshift_cluster_admin_password: "adminPassw0rd"
```

* Modify the number of users created for students.  By default, we create 20, but a rule of thumb is _number of workshop registrants + 5_:

##### Example:
```
vault_users_count: 20
```
The student accounts will be `user<#>`, with the same password as you set for the admin account.  You can assign each student a `user#` at the beginning of your workshop.
##### Example:

```
uid: user13
pwd: <same as admin user>
```

#### OPTIONAL
You can update the `vault_number_nodes` variable in `group_vars/all/vault.yml` to reflect the desired number of OpenShift nodes, if you want more than the default of `2`.  You probably don't need more nodes unless you have more than 20-25 students.


# Usage

**!Important** You must encrypt your group_vars/all/vault.yml before running your playbook.  You must add a vault_pass.txt to your home directory containing your password.

```
ansible-vault encrypt group_vars/all/vault.yml
```

You will be prompted to create a password and once complete, you can put this password is a file referenced in the ansible.cfg (vault_password_file = ~/.vault_pass.txt) file.  The current entry has a location of ~/.vault_pass.txt but you can change this at your discretion.

### Provision

Before you launch the provisioning script, copy the file `env.sh_example` to `env.sh`, and replace the placeholders with your AWS access key ID and secret access key.  Then, source that file, to place the variables into your environment.

```
source env.sh
AWS Keys exported
AWS_SECRET_ACCESS_KEY=01234567890abcdefghijlkmnopqrstuvwxyz!@#
AWS_ACCESS_KEY_ID=0123456789abcdefghij
```

* First, run the playbook to create your OpenShift instances
```
ansible-playbook 1_provision.yml
```
* Next, install OpenShift
```
ansible-playbook 2_load.yml
```
**[NOTE]:** A hidden directory contains the key pair for SSH access to instantiated host systems.

### Destroy
Once the workshop has been completed, you can tear down your OpenShift environment by running this playbook

```
ansible-playbook 3_unregister.yml
```
