---
layout: post
title: Automate - Publishing to Git Repositories
category: chef
tags: [chef, workflow, git, chef automate, automate]
summary: Ever wanted to copy your master to a remote master in Automate? Find out how!?
---

## Problem Summary
Would you like to browse code in an interface you are familiar with but isn't supported as a Source Code Provider for Chef Automate? Look no further! This guide will teach you how to publish your Git master branch to a remote Git master branch during the Publish phase of the Chef Automate pipeline.

## Publish Phase
The Publish phase is intended to be used to "publish" the code/artifacts from your pipeline to a location that other phases or stages can consume. The default publish recipe can be found in your project here `.delivery/build_cookbook/recipes/publish.rb`.

As you can see, the actions that take place in this phase are defined in the `delivery-truck` cookbook's [publish recipe](https://github.com/chef-cookbooks/delivery-truck/blob/master/recipes/publish.rb) and the places that your code/artifacts are published are defined in your `.delivery/config.json` file. For a full list of these options see [here](https://github.com/chef-cookbooks/delivery-truck#publish).


## Methods of Publishing to Git Repositories

### Method 1 - Using `config.json` and `delivery-secrets`
This method uses your `config.json` and encrypted data bags. Since this is already well documented in the `delivery-truck`'s [README](https://github.com/chef-cookbooks/delivery-truck/blob/master/README.md), if you would like to use this method see [here](https://github.com/chef-cookbooks/delivery-truck#publish)

> WARNING: This method uses encrypted data bags and by extension shared key encryption

### Method 2 - Using Chef Vault and `delivery_github`
Inside of `delivery-truck`'s publish recipe we can see that the custom resource `delivery_github` is [called](https://github.com/chef-cookbooks/delivery-truck#publish#L106). Since we include `delivery-truck` in our recipe this custom resource is available for us to use as well.

We can see from [line 107](https://github.com/chef-cookbooks/delivery-truck/blob/master/recipes/publish.rb#L107) that the `secrets` hash is passed into the `deploy_key` attribute. If we look at where this variable is set on [line 103](https://github.com/chef-cookbooks/delivery-truck/blob/master/recipes/publish.rb#L103) we can see it is set by the `get_project_secrets` method. Since we do not want to use shared key encryption or by extension encrypted data bags we will need to find another way to get this `deploy_key` value. One way of achieving this is via Chef Vault.

If you follow the recommendations in my [blog post about using Chef Vaults in Automate](http://blog.jerryaldrichiii.com/chef/2016/12/12/automate-using-chef-vaults-in-workflow.html) you will have everything you need to add this `deploy_key` attribute to your project's Chef Vault.

Just add the following key/value pairs to the `ent_name-org_name-project_name` vault under the `workflow-vaults` data bag on the Automate Chef Server:

```none
"git_repo_url": "ssh://git@some.git.host/<project-name>/<repo-name>",
"git_private_key": "PRIVATE KEY OF YOUR GIT USER",
```

Then you would add the following to your publish recipe (after the code snippet from my blog post):

```ruby
# INSERT CODE SNIPPET FROM BLOG POST HERE

delivery_github git_repo do
  deploy_key vault_data['git_private_key']
  branch node['delivery']['change']['pipeline']
  remote_url vault_data['git_repo_url']
  repo_path node['delivery']['workspace']['repo']
  cache_path node['delivery']['workspace']['cache']
  action :push
end
```

Doing so will allow you to publish your code to a Git repository without the need of relying on shared key encryption.

## Extra Resources
  - [Chef Vault](https://github.com/chef/chef-vault)
  - [delivery-truck](https://github.com/chef-cookbooks/delivery-truck)
