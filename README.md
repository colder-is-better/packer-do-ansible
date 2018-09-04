# An example of using Packer with the DigitalOcean builder

The snapshot image we create has Ubuntu Server 18.04 LTS and nginx preconfigured to serve a custom domain. For the provisioning of the image we use Ansible. Before we run the Packer builder for the first time we need to edit `template.json` and replace `PUT_YOUR_DO_API_TOKEN_HERE` with our [DigitalOcean API token](https://www.digitalocean.com/docs/api).

__Build__ the image:

	packer build template.json

__List__ all your images:

	doctl compute image list-user

`doctl` is a [command line interface for the DigitalOcean API](https://github.com/digitalocean/doctl).

__Create__ a new droplet (VM) like this:

	doctl compute droplet create delta \
	--image 12345678 --region fra1 \
	--size s-2vcpu-2gb --ssh-keys ssh_fingerprint

`12345678` should be replaced with the ID of the snapshot image (look at the output of previous command), and `ssh_fingerprint` should be replaced with one of the SSH public keys you have uploaded to your DigitalOcean account. To list the fingerprints of all uploaded keys, type `doctl compute ssh-key list`. You may also choose a different name for your droplet (we chose `delta`).

__Connect__ to the root account of the new droplet:

	doctl compute ssh delta

__Make__ sure nginx is up and running. Just fire up your favorite web browser and visit `http://your.droplet.ip.addr`. You should see a page with a short message: [þetta reddast!](http://www.bbc.com/travel/story/20180603-the-unexpected-philosophy-icelanders-live-by)

At any time you wish to run the Ansible provisioner against the droplet, e.g., to refresh repositories and apply any updates, make sure `my.droplet.ip.address` in `hosts` file is replaced with the actual IP or FQDN of the droplet, and then type:

	ansible-playbook -i hosts provision.yml

Should you decide to __delete the droplet__, type `doctl compute droplet list`, take a note of the droplet ID, then type something like this:

	doctl compute droplet delete 123456789

Finally, to __delete the snapshot image__ itself just type something like this:

	doctl compute image delete 12345678

_This repository has been created to complement an extensive article on [how to programmatically manage resources in the DigitalOcean cloud](https://deltahacker.gr/?p=18089), written for [deltaHacker magazine](https://deltahacker.gr) (text in Greek, active subscription may be required)._
