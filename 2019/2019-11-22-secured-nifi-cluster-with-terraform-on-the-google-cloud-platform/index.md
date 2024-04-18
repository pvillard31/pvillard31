---
title: "Secured NiFi cluster with Terraform on the Google Cloud Platform"
date: "2019-11-22"
categories: 
  - "apache-nifi"
  - "google-cloud-platform"
tags: 
  - "google-cloud"
  - "nifi-ca"
  - "oidc"
  - "terraform"
coverImage: "gcpnifi.png"
---

This story is a follow up of [this previous story](https://pierrevillard.com/2019/08/21/nifi-with-oidc-using-terraform-on-the-google-cloud-platform/) about deploying a single secured NiFi instance, configured with OIDC, using Terraform on the Google Cloud Platform. This time it’s about deploying a secured NiFi cluster.

In this story, we’ll use Terraform to quickly:

- **deploy a NiFi CA server** as a convenient way to generate TLS certificates
- **deploy an external ZooKeeper instance** to manage cluster coordination and state across the nodes
- **deploy X secured NiFi instances clustered together**
- **configure NiFi to use OpenID connect** for authentication
- **configure an HTTPS load balancer with Client IP affinity** in front of the NiFi cluster

_Note_ — I assume you have a domain that you own ([you can get one with Google](https://domains.google/)). It will be used to map a domain to the web interface exposed by the NiFi cluster. In this post, I use my own domain: pierrevillard.com and will map nifi.pierrevillard.com to my NiFi cluster.

_Disclaimer_ — the below steps should not be used for a production deployment, it can definitely get you started but I’m just using the below to start a secured cluster (there is no configuration that one would expect for a production setup such as a clustered Zookeeper, disks for repositories, etc).

**If you don’t want to read the story and want to get straight into the code,** [**it’s right here**](https://github.com/pvillard31/nifi-gcp-terraform/tree/master/gcp-cluster-secured-nifi-oidc)**!**

* * *

**What is Terraform?**

[Terraform](https://www.terraform.io/intro/index.html) is a tool for building, changing, and versioning infrastructure safely and efficiently. Terraform can manage existing and popular service providers as well as custom in-house solutions.

Configuration files describe to Terraform the components needed to run a single application or your entire datacenter. Terraform generates an execution plan describing what it will do to reach the desired state, and then executes it to build the described infrastructure. As the configuration changes, Terraform is able to determine what changed and create incremental execution plans which can be applied.

The infrastructure Terraform can manage includes low-level components such as compute instances, storage, and networking, as well as high-level components such as DNS entries, SaaS features, etc.

* * *

**What is NiFi?**

[Apache NiFi](https://nifi.apache.org/) is an easy to use, powerful, and reliable system to process and distribute data. Apache NiFi supports powerful and scalable directed graphs of data routing, transformation, and system mediation logic. In simpler words, Apache NiFi is a great tool to collect and move data around, process it, clean it and integrate it with other systems. As soon as you need to bring data in, you want to use Apache NiFi.

* * *

**Why ZooKeeper?**

<figure>

![](images/c5fe5-1ovol6a5dsep8un7jpqos4w.png)

<figcaption>

Apache NiFi clustering

</figcaption>

</figure>

Best is to [refer to the documentation](https://nifi.apache.org/docs/nifi-docs/html/administration-guide.html#clustering), but, in short… NiFi employs a Zero-Master Clustering paradigm. Each node in the cluster performs the same tasks on the data, but each operates on a different set of data. One of the nodes is automatically elected (via Apache ZooKeeper) as the Cluster Coordinator. All nodes in the cluster will then send heartbeat/status information to this node, and this node is responsible for disconnecting nodes that do not report any heartbeat status for some amount of time. Additionally, when a new node elects to join the cluster, the new node must first connect to the currently-elected Cluster Coordinator in order to obtain the most up-to-date flow.

* * *

### OAuth Credentials

First step is to create the OAuth Credentials (at this moment, this cannot be done using Terraform).

- Go in your GCP project, APIs & Services, Credentials.
- Click on Create credentials, OAuth client ID. Select Web application.
- Give a name like “NiFi”. For Authorized JavaScript origins, use your own domain. I’m using: [https://nifi.pierrevillard.com](https://nifi.pierrevillard.com:8443./). For Authorized redirect URIs, I’m using: [https://nifi.pierrevillard.com/nifi-api/access/oidc/callback](https://nifi.pierrevillard.com:8443/nifi-api/access/oidc/callback). Please adapt with your own domain (note there is no port as we'll use the load balancer to access the cluster)
- Click Create

<figure>

![](images/15084-181d-3oivool57hbq8e-b1w.png)

<figcaption>

Create the OAuth credentials

</figcaption>

</figure>

Once the credentials are created, you will get a client ID and a client secret that you will need in the Terraform variables.

By creating the credentials, your domain will be automatically added to the list of the “Authorized domains” in the OAuth consent screen configuration. It protects you and your users by ensuring that OAuth authentication is only coming from authorized domains.

### Download the NiFi binaries in Google Cloud Storage

In your GCP project, create a bucket in Google Cloud Storage. We are going to use the bucket to store the Apache NiFi & ZooKeeper binaries (instead of downloading directly from the Apache repositories at each deployment), and also as a way to retrieve the certificates that we’ll use for the HTTPS load balancer.

_Note — you’ll need Apache ZooKeeper 3.5.5+._

You can download the binaries using the below links:

- [Apache NiFi](https://nifi.apache.org/download.html)
- [Apache ZooKeeper](https://zookeeper.apache.org/releases.html#download)

Here is what it looks like:

<figure>

![](images/bf4d4-1ll5i86yzqze-xp6dyf0jsq.png)

<figcaption>

Content of the bucket in Google Cloud Storage

</figcaption>

</figure>

_Note — you’ll need to use the NiFi Toolkit version 1.9.2_

### Deploy NiFi with Terraform

Once you have completed the above prerequisites, installing your NiFi cluster will only take few minutes. Open your Google Cloud Console in your GCP project and run:

<figure>

````
git clone https://github.com/pvillard31/nifi-gcp-terraform.git
cd nifi-gcp-terraform/gcp-cluster-secured-nifi-oidc
/bin/sh deploy.sh <gcp-project-id> <gcs-bucket>
````

<figcaption>

Deploy script

</figcaption>



</figure>

If you execute the above commands, you’ll be prompted for the below informations. However, if you don’t want to be prompted, you can directly update the _variables.tf_ file with your values to deploy everything.

Variables to update:

- **project** // GCP Project ID
- **nifi-admin** // Google mail address for the user that will be the initial admin in NiFi
- **san** // FQDN of the DNS mapping for that will be used to access NiFi. Example: nifi.example.com
- **proxyhost** // FQDN:port that will be used to access NiFi. Example: nifi.example.com:8443
- **ca\_token** // The token to use to prevent MITM between the NiFi CA client and the NiFi CA server (must be at least 16 bytes long)
- **oauth\_clientid** // OAuth Client ID
- **oauth\_secret** // OAuth Client secret
- **instance\_count** // Number of NiFi instances to create
- **nifi\_bucket** // Google Cloud Storage bucket containing the binaries

Here is what it looks like on my side (after updating the _variables.tf_ file):

<figure>

````
pvillard@cloudshell:~/nifi-gcp-terraform/gcp-cluster-secured-nifi-oidc (nifi-dev-project)$ /bin/sh deploy.sh nifi-dev-project nifi_bin
Updated property [core/project].

Initializing the backend...

Initializing provider plugins...

The following providers do not have any version constraints in configuration,
so the latest version was installed.

To prevent automatic upgrades to new major versions that may contain breaking
changes, it is recommended to add version = "..." constraints to the
corresponding provider blocks in configuration, with the constraint strings
suggested below.

* provider.google: version = "~> 2.13"

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
google_compute_network.default: Creating...
google_compute_network.default: Still creating... [10s elapsed]
google_compute_network.default: Creation complete after 17s [id=nifi-network]
google_compute_subnetwork.default: Creating...
google_compute_subnetwork.default: Still creating... [10s elapsed]
google_compute_subnetwork.default: Creation complete after 18s [id=europe-west1/nifi-network]
google_compute_instance.nifi-ca: Creating...
google_compute_instance.nifi-ca: Still creating... [10s elapsed]
google_compute_instance.nifi-ca: Creation complete after 10s [id=nifi-ca]

Apply complete! Resources: 3 added, 0 changed, 0 destroyed.
CommandException: One or more URLs matched no objects.
... (waiting for Load Balancer certificates to be generated)
CommandException: One or more URLs matched no objects.
gs://nifi_bin/key.pem
Copying gs://nifi_bin/key.pem...
/ [1 files][  1.8 KiB/  1.8 KiB]
Operation completed over 1 objects/1.8 KiB.
Copying gs://nifi_bin/certs.pem...
/ [1 files][  2.7 KiB/  2.7 KiB]
Operation completed over 1 objects/2.7 KiB.
google_compute_network.default: Refreshing state... [id=nifi-network]
google_compute_subnetwork.default: Refreshing state... [id=europe-west1/nifi-network]
google_compute_instance.nifi-ca: Refreshing state... [id=nifi-ca]
google_compute_https_health_check.nifi-healthcheck: Creating...
google_compute_firewall.allow-ssh: Creating...
google_compute_firewall.allow-https: Creating...
google_compute_ssl_certificate.nifi-lb-cert: Creating...
google_compute_firewall.allow-internal: Creating...
google_compute_instance.zookeeper: Creating...
google_compute_https_health_check.nifi-healthcheck: Creation complete after 4s [id=nifi-healthcheck]
google_compute_ssl_certificate.nifi-lb-cert: Creation complete after 5s [id=nifi-lb-cert]
google_compute_firewall.allow-ssh: Creation complete after 9s [id=allow-ssh]
google_compute_firewall.allow-https: Still creating... [10s elapsed]
google_compute_firewall.allow-internal: Still creating... [10s elapsed]
google_compute_instance.zookeeper: Still creating... [10s elapsed]
google_compute_instance.zookeeper: Creation complete after 12s [id=zookeeper]
google_compute_instance.nifi[4]: Creating...
google_compute_instance.nifi[2]: Creating...
google_compute_instance.nifi[0]: Creating...
google_compute_instance.nifi[1]: Creating...
google_compute_instance.nifi[3]: Creating...
google_compute_instance.nifi[5]: Creating...
google_compute_firewall.allow-https: Creation complete after 17s [id=allow-https]
google_compute_firewall.allow-internal: Creation complete after 17s [id=allow-internal]
google_compute_instance.nifi[2]: Creation complete after 7s [id=nifi-3]
google_compute_instance.nifi[4]: Still creating... [10s elapsed]
google_compute_instance.nifi[0]: Still creating... [10s elapsed]
google_compute_instance.nifi[3]: Still creating... [10s elapsed]
google_compute_instance.nifi[1]: Still creating... [10s elapsed]
google_compute_instance.nifi[5]: Still creating... [10s elapsed]
google_compute_instance.nifi[1]: Creation complete after 11s [id=nifi-2]
google_compute_instance.nifi[5]: Creation complete after 11s [id=nifi-6]
google_compute_instance.nifi[0]: Creation complete after 12s [id=nifi-1]
google_compute_instance.nifi[4]: Creation complete after 12s [id=nifi-5]
google_compute_instance.nifi[3]: Creation complete after 12s [id=nifi-4]
google_compute_instance_group.nifi-ig: Creating...
google_compute_instance_group.nifi-ig: Creation complete after 7s [id=europe-west1-d/nifi-ig]
google_compute_backend_service.nifi-backend: Creating...
google_compute_backend_service.nifi-backend: Creation complete after 9s [id=nifi-backend]
google_compute_url_map.nifi-url-map: Creating...
google_compute_url_map.nifi-url-map: Creation complete after 4s [id=nifi-url-map]
google_compute_target_https_proxy.nifi-target-proxy: Creating...
google_compute_target_https_proxy.nifi-target-proxy: Creation complete after 4s [id=nifi-target-proxy]
google_compute_global_forwarding_rule.nifi-lb: Creating...
google_compute_global_forwarding_rule.nifi-lb: Still creating... [10s elapsed]
google_compute_global_forwarding_rule.nifi-lb: Creation complete after 17s [id=nifi-lb]

Apply complete! Resources: 17 added, 0 changed, 0 destroyed.
Removing gs://nifi_bin/key.pem...
/ [1 objects]
Operation completed over 1 objects.
Removing gs://nifi_bin/certs.pem...
/ [1 objects]
Operation completed over 1 objects.
````

<figcaption>

Execution of the deploy script

</figcaption>



</figure>

### Explanations

The first step is to deploy the NiFi Toolkit on a single VM to run the CA server that is used to generate certificates for the nodes and the load balancer. Once the CA server is deployed, a certificate is generated for the load balancer and pushed to the Google Cloud Storage bucket.

The script you started is waiting until the load balancer certificate files are available on GCS. Once the files are available, files are retrieved locally to execute the remaining parts of the Terraform template. It will deploy the ZooKeeper instance as well as the NiFi instances and the load balancer in front of the cluster. All the configuration on the NiFi instances is done for you. Once the script execution is completed, certificates files are removed (locally and on GCS).

### After 5 minutes or so…

The load balancer has been created and you can retrieve the public IP of the load balancer:

<figure>

![](images/98940-1au5ax9we5uclc-du8mztuw.png)

<figcaption>

Retrieve the external public IP of the HTTPS load balancer

</figcaption>

</figure>

You can now update the DNS records of your domain to add a DNS record of type A redirecting nifi.pierrevillard.com to the load balancer IP.

I can now access the NiFi cluster using [https://nifi.pierrevillard.com](https://nifi.pierrevillard.com) and authenticate on the cluster using the admin account email address I configured during the deployment.

Here is my 6-nodes secured NiFi cluster up and running:

<figure>

![](images/9bf24-1hrzlw72ag4dlw7ex3ei9qa.png)

<figcaption>

6-nodes secured NiFi cluster

</figcaption>

</figure>

<figure>

![](images/52459-13-fi1okyctvuvm8hxv_bsw.png)

<figcaption>

6 nodes with the elected primary and coordinator nodes

</figcaption>

</figure>

I can now update the authorizations and add additional users/groups.

_Note — you could use Google certificates instead of the ones generated with the CA server to remove the warnings about untrusted certificate authority._

### Cleaning

To destroy all the resources you created, you just need to run:

**terraform destroy -auto-approve**

As usual, thanks for reading, feel free to ask questions or comment this post.
