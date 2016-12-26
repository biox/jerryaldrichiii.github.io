---
layout: post
title: Automate - Deploying to Multiple Chef Servers
category: chef
tags: [chef, workflow, git, chef automate, automate, deploy]
summary: Have more than one Chef server/organization? Want to deploy to those? Read this.
---

## Problem Summary
By default Automate is setup to deploy to a single Chef organization/server. This deployment usually occurs during the Publish phase. This is fine for most users if all of your nodes are in the same Chef organization as your Automate infrastructure. Additionally, if version pinning is done correctly then there is no issue with this model.

Some other users would like to have the Delivered stage deploy a cookbook to a separate Chef organization instead of converging on their production infrastructure. This guide is directed to that group of users.

## Deploy Phase
The Deploy phase in Automate is intended to "deploy" the artifact created in the Publish phase to node(s) prior to testing. For the purpose of this guide we are only concerned with what happens during the Deploy phase of the Delivered stage.

## Summary of Approach
The following items have to occur prior to deploying cookbooks to other Chef organizations:

  - A user must exist on each Chef organization you want to deploy to
  - The private keys of those users must be stored in a secure way
  - The private keys of those users must be written to disk
  - Knife configuration files (`knife.rb`) must be created and written to disk

## Creating Keys For Chef
Before we can access another Chef organization, we must have have a user and a key that has been granted access to that organization. For example, you might create a `production` user if you had a 'production' organization (similar to the automate user/automate org coupling that already exists).

### Creating a Chef User
Chef users can be created either from your workstation with `knife` or on the Chef Server with `chef-server-ctl`. This guide covers the latter.

Run the following command substituting your values where appropriate

```none
chef-server-ctl user-create \
  USER_NAME \
  FIRST_NAME \
  [OPTIONAL_MIDDLE_NAME] \
  LAST_NAME \
  EMAIL \
  'PASSWORD' \
  --filename /path/to/output/key/keep_me.pem
```

> After creating this user make sure you take note of where the key is saved. It will be needed later.

### Adding the Chef User to an Organization
After creating the user above it needs to be added to the organization(s) you wish to deploy to.

```none
chef-server-ctl org-user-add USER_NAME ORGANIZATION_NAME
```

### Safely Storing Your Key(s)
Secrets are hard. There are many solutions on how to store your private keys. One method is covered in my blog post about [using Chef Vault in Workflow](http://blog.jerryaldrichiii.com/chef/2016/12/12/automate-using-chef-vaults-in-workflow.html).

Necessary secrets:
*can be an Enterprise/Org/Project secret according to your need, see above blog post for reference*

```none
...
"deploy": {
  "chef_servers": [
    {
      "url": "https://my.chef.server/organizations/my_org",
      "user": "chefuser",
      "key": "RSA_PRIVATE_KEY\nALL_ONE_LINE\nWITH_NEWLINES_ESCAPED"
    },
    {
      "url": "https://my.chef.server/organizations/my_other_org",
      "user": "anotherchefuser",
      "key": "RSA_PRIVATE_KEY\nALL_ONE_LINE\nWITH_NEWLINES_ESCAPED"
    }
  ]
},
...
```

### The Deploy Phase
Place the following code below `include_recipe 'delivery-truck::deploy'` in `.delivery/build_cookbook/recipes/deploy.rb` within the pipeline you're working with.

```ruby
case workflow_stage

# Ensure the following actions only occur in the Delivered stage
when 'delivered'

  # Get a hash of the data from the appropriate Chef Vault
  vault_data = workflow_vault_data

  # Iterate through Chef Servers
  vault_data['deploy']['chef_servers'].each do |server_info|
    # Set file paths inside project cache
    client_key_path = File.join(workflow_workspace_cache, 'delete_me.pem')
    knife_rb_path = File.join(workflow_workspace_cache, 'knife.rb')

    # Create key for Knife to use (gets overwritten by each deploy)
    file client_key_path do
      content server_info['key']
      sensitive true
      action :create
    end

    # Create knife.rb file (gets overwritten by each deploy)
    file knife_rb_path do
      # Set file content and strip leading white space
      content <<-EOF.gsub(/^\s+/, '')
        log_location             STDOUT
        node_name                "#{server_info['user']}"
        client_key               "#{client_key_path}"
        chef_server_url          "#{server_info['url']}"
        trusted_certs_dir        "/etc/chef/trusted_certs"
      EOF
      action :create
    end

    # Create the upload directory where cookbooks to be uploaded will be staged
    cookbook_vendor = File.join(workflow_workspace_cache, 'cookbook-vendor')
    directory cookbook_vendor do
      recursive true
      # We delete the cookbook upload staging directory each time to ensure we
      # don't have out-of-date cookbooks hanging around from a previous deploy.
      action [:delete, :create]
    end

    # Perform a `berks install` and set path to vendor directory
    execute "do berks install in #{workflow_change_project} cookbook" do
      command 'berks install'
      live_stream true
      environment BERKSHELF_PATH: cookbook_vendor
      cwd workflow_workspace_repo
    end

    # Perform a `berks upload` using the Knife config generated earlier
    execute "do berks upload in #{workflow_change_project} cookbook" do
      command 'berks upload'
      live_stream true
      environment(
        BERKSHELF_CHEF_CONFIG: knife_rb_path,
        BERKSHELF_PATH: cookbook_vendor
      )
      cwd workflow_workspace_repo
    end

    # Ensure keys are delete after deploy is done
    [client_key_path, knife_rb_path].each do |file_path|
      file file_path do
        action :delete
      end
    end
  end
end
```

After adding to above code to your deploy phase, give it a test run! After a successful delivery you should see your new cookbook(s) arrive in your second Chef organization.

## Extra Resources
  - [Using Chef Vaults in Workflow](http://blog.jerryaldrichiii.com/chef/2016/12/12/automate-using-chef-vaults-in-workflow.html)
