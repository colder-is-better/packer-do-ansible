# Using the DO Packer builder with an Ansible provisioner

We programmatically create a snapshot image in the cloud of [DigitalOcean](http://digitalocean.com), with Ubuntu Server 18.04 LTS and nginx preconfigured to serve a custom domain. For provisioning we use [Ansible](https://www.ansible.com). Before we run the Packer builder we open `template.json` and make sure `PUT_YOUR_DO_API_TOKEN_HERE` is replaced with our [DigitalOcean API token](https://www.digitalocean.com/docs/api). Please note that said API token should have the __Write Scope__ enabled.

Here is how you can create your own snapshot image, spawn a new droplet off of it, then delete the droplet and also the snapshot. 

__Build__ the image:

	packer build template.json

__List__ all your images in the DigitalOcean cloud:

	doctl compute image list-user

`doctl` is a [command line interface for the DigitalOcean API](https://github.com/digitalocean/doctl).

__Create__ a new droplet (VM) like this:

	doctl compute droplet create delta \
	--image img_id --region fra1 \
	--size s-2vcpu-2gb --ssh-keys ssh_fprint

`img_id` should be replaced with the ID of the snapshot image (look at the output of previous command), and `ssh_fprint` should be replaced with the fingerprint of one of the SSH public keys you have stored in your DigitalOcean account. To list the fingerprints of all stored keys, type `doctl compute ssh-key list`. You may also choose a different name for your droplet (we chose `delta`).

__Connect__ to the root account of the new droplet:

	doctl compute ssh delta

__Make__ sure nginx is indeed up and running. Just fire up your favorite web browser and visit `http://your.droplet.ip.addr`. You should see a page with a short message: [þetta reddast!](http://www.bbc.com/travel/story/20180603-the-unexpected-philosophy-icelanders-live-by)

Any time you wish to run the Ansible provisioner against the droplet, e.g., to refresh repositories and apply any updates, make sure `my.droplet.ip.address` in `hosts` file is replaced with the actual IP or FQDN of the droplet, then type:

	ansible-playbook -i hosts provision.yml

Should you decide to __delete the droplet__, type `doctl compute droplet list`, take a note of the droplet ID, then type something like this:

	doctl compute droplet delete droplet_id

Finally, to __delete the snapshot image__ itself, just type something like this:

	doctl compute image delete img_id

_This repository has been created to complement an extensive article on [how to programmatically manage resources in the DigitalOcean cloud](https://deltahacker.gr/?p=18089), written for [deltaHacker magazine](https://deltahacker.gr) (text in Greek, active subscription may be required)._
