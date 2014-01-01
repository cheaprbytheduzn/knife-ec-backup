# knife EC backup

# Description

This is an UNOFFICIAL and EXPERIMENTAL knife plugin intended to back up and restore an entire Enterprise Chef / Private Chef server, preserving the data in an intermediate, editable text format.

# Requirements

This knife plugin currently requires the Knife-Essentials gem to be installed in the same gemset. This requirement is currently hosted here:

    https://github.com/jkeiser/knife-essentials

# Installation

## Chef Server Install (Recommended)
This will install the plugin directly on the Chef Server:

    /opt/opscode/embedded/bin/gem install knife-essentials
    /opt/opscode/embedded/bin/gem install knife-ec-backup

## Build from source
Clone the git repository and run the following from inside:

    gem build knife-ec-backup.gemspec
    gem install knife-ec-backup-1.1.3.gem

# Configuration

## Permissions
Note that most users in an EC installation lack the permissions to pull all of the data from all organizations and other users.
This plugin **REQUIRES THE PIVOTAL KEY AND WEBUI KEY** from the Chef Server.
It is recommended that you run this from a frontend Enterprise Chef Server, you can use --user and --key to pass the pivotal information along.

# Subcommands

## knife ec backup DEST_DIR (options)

*Options*

  * `--concurrency`:
    Maximum number of simultaneous requests to send (default: 10)
  * `--webui-key`:
    Used to set the path to the WebUI Key (default: /etc/opscode/webui_priv.pem)
  * `--skip-useracl`:
    Whether to skip downloading User ACLs.  This is required for EC 11.0.0 and lower (default: false)
  * `--skip-version-check'`:
    Whether to skip checking the Chef Server version.  This will also skip any auto-configured options (default: false)

# Usage Example

## Create a Working Directory Structure

```
mkdir -p transfer/.chef
mkdir transfer/backup_dir
```

## Populate your transfer/.chef/knife.rb config file

```ruby
transfer_repo = File.expand_path('..', File.dirname(__FILE__))
chef_server_root      'https://CHEF-SEVER-ROOT'
node_name             'pivotal'
client_key            "#{transfer_repo}/.chef/pivotal.pem"
repo_mode             'hosted_everything'
versioned_cookbooks   true
chef_repo_path        transfer_repo
cookbook_path         nil
```

Enter the root of your Chef server as applicable.

## Install Pivotal and Webui Keys

Check /etc/opscode on your Chef server and copy the keys to:

```
.chef/pivotal.pem
.chef/webui_priv.pem
```

## Execute a Backup

From the transfer dir:

```bash
/opt/opscode/embedded/bin/knife ec backup backup_dir --webui-key .chef/webui_priv.pem
```

## Troubleshooting

Add the `-VV` option to enable debug logging.

# Output Directory Structure

The format of the repository is based on the `knife-essentials` (`knife download`) format and looks like this:

    users
      <name>.json
    user_acls
      <name>.json
    organizations
      <orgname>
        acls
          <type>
            <name>.json
        clients
          <name>.json
        containers
          <name>.json
        cookbooks
          <name>-<version>
        data_bags
          <bag name>
            <item name>
        environments
          <name>.json
        groups
          <name>.json
        nodes
          <name>.json
        roles
          <name>.json
        org.json
        members.json
        invitations.json

This compares very closely with the "knife download /" from an OSC server:

    clients
      <name>.json
    cookbooks
      <name>-<version>
    data_bags
      <bag name>
        <item name>
    environments
      <name>.json
    nodes
      <name>.json
    roles
      <name>.json
    users
      <name>.json>

## knife ec restore DEST_DIR (options)

*Options*

  * `--concurrency`:
    Maximum number of simultaneous requests to send (default: 10)
  * `--webui-key`:
    Used to set the path to the WebUI Key (default: /etc/opscode/webui_priv.pem)
  * `--overwrite-pivotal`:
    Whether to overwrite pivotal's key.  Once this is done, future requests will fail until you fix the private key (default: false)
  * `--skip-useracl`:
    Whether to skip downloading User ACLs.  This is required for EC 11.0.0 and lower (default: false)
  * `--skip-version-check'`:
    Whether to skip checking the Chef Server version.  This will also skip any auto-configured options (default: false)

Restores all data from a repository to an Enterprise Chef / Private Chef server.

# TODO

* Ensure easy installation into embedded ruby gemset on Chef Server.
* Remove requirement for Knife Essentials gem to be installed.
* This plugin does **NOT** currently backup user passwords.  **They will have to be reset after a restore.**
* This plugin does **NOT** currently restore custom user ACLs.  **It will revert back to default ACLs on a restore.**
