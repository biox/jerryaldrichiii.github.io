---
layout: post
title: Automate - Using Chef Vaults in Workflow
category: chef
tags: [chef, workflow]
summary: Secrets are hard. Shared key encryption is bad. Use Chef Vault.
---

## What is Chef Vault
Chef Vault is a solution for securing data used during a Chef Client run. Chef Vault solves the shared key encryption problem present in encrypted data bags.

Currently, encrypted data bags use shared key encryption; this means that both the key to encrypt and decrypt the data are the same. This leads to security issues due to the fact that this single key must be shared with anyone who wants to use the data.

Chef Vault solves the shared key encryption problem by using the already present trust relationship between Chef clients and the Chef Server. By using each client's public key to encrypt the data and allowing the clients to decrypt the data using their secret private key at no point do sensitive keys need to be transmitted.

For more info on how Chef Vaults work see [here](https://github.com/chef/chef-vault).

---

## Using Chef Vaults in Chef Automate Workflow
The method demonstrated in this post uses a data bag called `workflow-vaults` to hold all of the Chef Vaults used by Workflow projects.

The vaults inside of `workflow-vaults` are created with a specific naming and scoping scheme to enable the following:

  - Using the `*-slug` helper methods in [delivery-sugar](https://github.com/chef-cookbooks/delivery-sugar#workflow_project_slug) to load vaults
  - Sharing data within the Enterprise, Organization, and Project
  - Reducing variable duplication through merging down to a single Ruby hash

---

## Naming Scheme and Scope
Vaults should be named and data be scoped as follows:

| Vault <br> Name                           | Enterprise <br> Scope | Organization <br> Scope | Project <br> Scope |
|------------------------------------------ | --------------------- | ----------------------- | ------------------ |
| `#{ent_name}`                             | Y                     | Y                       | Y                  |
| `#{ent_name}-#{org_name}`                 |                       | Y                       | Y                  |
| `#{ent_name}-#{org_name}-#{project_name}` |                       |                         | Y                  |

---

## Creating the Chef Vaults
There are a few options for creating Chef Vaults. You can set your `EDITOR` environment variable and type the JSON at creation, pass the JSON in as a string, or create JSON files on disk and reference them.

I personally prefer creating the files on disk and referencing them. To use this method do the following:

### Create your directory structure

```none
workflow-vaults
├── my_ent.json
├── my_ent-my_org.json
├── my_ent-my_org-project1.json
└── my_ent-my_org-project2.json
```

### Populate your JSON files (EXAMPLE)

```none
{
  "id":"my_ent-my_org-project1",
  "secret_data": "something secret",
  "other_data": "something else"
}
```

> NOTE: Keys with the same name in multiple vaults will be merged in such a way that Project level data will overwrite Organization data and Organization data will overwrite Enterprise data

### Create your vaults using `knife vault create` (EXAMPLE)

```none
knife vault create \
  workflow-vaults \
  my_ent-my_org-project1 \
  -J '/path/to/my_ent-my_org-project1.json' \
  -A 'delivery,jerry' \
  -S 'tags:delivery-job-runner' \
  -M client
```

| Argument                          | Purpose                                                   |
| --------------------------------- | --------------------------------------------------------- |
| `workflow-vaults`                 | Selects the data bag that will contain our vaults         |
| `my_ent-my_org-my_project`        | Sets the Chef Vault name in `workflow-vaults`             |
| `-A 'delivery,jerry'`             | Comma separated list of Admins to grant access            |
| `-S 'tags:delivery-job-runner'`   | SOLR search that gives access to all Workflow runners     |
| `-M client`                       | Enables creating vaults on Chef Server instead of locally |

> NOTE: After creating your vaults it is recommended that you delete the JSON files from disk

---

## Using Chef Vaults in your build_cookbook
From any recipe in your build_cookbook you can now use the below code to access your vault data:

```ruby
# Load the `chef-vault` gem
require 'chef-vault'

# Load a Chef config that has rights to view workflow-vaults
Chef::Config.from_file(automate_knife_rb)

# Compile list of Vault items using `delivery-sugar` helper methods
vault_items = [
  workflow_change_enterprise,
  workflow_organization_slug,
  workflow_project_slug
]

# Populate a list of hashes with empty hashes for non-existent vaults
vault_data_list = []
vault_items.each do |item|
  vault_data_list.push(
    begin
      ChefVault::Item.load('workflow-vaults', item)
    rescue ChefVault::Exceptions::KeysNotFound
      {}
    end
  )
end

# Merge each of the hashes above into a single hash
vault_data = vault_data_list.inject(&:merge)

# Raise an error if no data is found
if vault_data.empty?
  raise 'No Chef Vaults found in \'workflow-vaults\' that match naming standard'
end
```

> NOTE: The above code only works if you added `-A 'delivery'` when creating your Chef Vaults

## Extra Resources

  - [Chef Vault GitHub](https://github.com/chef/chef-vault)
  - [Chef Vault Blog Post](https://blog.chef.io/2016/01/21/chef-vault-what-is-it-and-what-can-it-do-for-you/)
  - [Knife Examples](https://github.com/chef/chef-vault/blob/master/KNIFE_EXAMPLES.md)
