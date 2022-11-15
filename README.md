# Lab Ingress

This repository hosts a set of resources to deploy an HAProxy and Nginx/Let's Encrypt set of containers to provide a reverse proxy for a set of services.

Primarily this allows easy fronting of services across a set of hosts/ports/names with a single IP and SSL certificate.

## Host Requirements

- Podman
- FirewallD

The Ansible Playbook will install all the latest versions of these packages on the target hosts.

## Using this repository

First off, the configuration in this repo is mine which means it's not likely to be usable by you - and you don't have edit access to this repo more than likely.  So, you'll need to fork this repo and then clone your forked repo to your local machine.

1. [Fork this repo](https://github.com/kenmoini/lab-ingress/fork)
2. Clone your forked repo to your local machine - `git clone https://github.com/YOUR_USERNAME/lab-ingress.git`.
3. If you'll be running the Playbook manually with the CLI then set up your inventory file - see the [examples/inventory](examples/inventory) file for an example of how to do this.  If you plan on using this with Ansible Tower you can skip creating the inventory file.
4. Now create your site configuration variable file - you can find the example file in the [examples/site_config.yml](examples/site_config.yml) file.  You'll need to edit this file to match your inventory and the services running on each host.  Write your customized version to `vars/site_config.yml`.
5. Now you can run the Playbook to deploy the Ingress services to your hosts.  If you're using the CLI, you can run the Playbook like this:

```bash=
ansible-playbook -i inventory deploy.yml
```

> If you're using Ansible Tower or the AAP2 Controller, continue to the next step.

## Setting Up Ansible Tower for GitOps-y goodness

Ideally as soon as you commit a change to the `main` branch of this repo, the changes will be synced to the actual services running on the hosts.  To do this, we'll use Ansible Tower to run the `deploy.yml` playbook.  Assuming you already have Ansible Tower or the AAP2 Controller installed and ready to go, you can follow these steps to set things up:

1. Create a **Credential(s)** for the systems that you will be connecting to.  You can use the `Machine` credential type for this.  You can also use the `Source Control` credential type for the Git repo that you'll be using for the configuration files.
2. Create an **Inventory** for the hosts that you'll be deploying the DNS services to.
3. Add the **Hosts** to the **Inventory**.
4. Create a **Project** for your fork of this Git repo.  I like to keep the names the same as the repo, so I named mine `lab-ingress`.  Make sure to check the `Update Revision on Launch` box.
5. Create a **Template** for the `deploy.yml` Playbook, add the Credentials you'll need to connect to the hosts.  Important: Make sure to check the `Enable Webhook` box.  This will allow you to trigger the Playbook from a GitHub/GitLab webhook.
6. Once the Template is created, you can get the `Webhook URL` and the `Webhook Key` from the **Details** page of the Template.  You'll need these to set up the Webhook in GitHub/GitLab.

Assuming you'll be using this with GitHub, you can follow these steps to set up the Webhook: https://docs.ansible.com/ansible-tower/latest/html/userguide/webhooks.html

> With this, now when you perform a `git push` to the repo to update your Ingress configuration, the Playbook will be triggered and the changes will be automatically deployed to the hosts!

## Let's Encrypt Certificates

Currently the method to leverage LE is kind of rudimentary - this will be improved upon soon.