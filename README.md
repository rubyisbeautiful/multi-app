# ansible-multi-app
[![Build Status](https://semaphoreci.com/api/v1/rubyisbeautiful/multi-app/branches/master/badge.svg)](https://semaphoreci.com/rubyisbeautiful/multi-app)

## Overview

This role simplifies the pattern of downloading and installing 3rd party
software, particularly the kind that does not require compilation but that
simply runs "out of the box," such as Java, Tomcat, Node.js, etc.

There are many configurable options -- see defaults/main.yml for additional
variables.


## Basic Usage

You must set the following variables for each app:

```
multiapp_app_installations:
  app:
    - app_version: 1.8.0_121-32bit
      app_extracted_dir: jdk1.8.0_121-32bit
      app_artifact: http://some.internal.repo/mirrors/jdk1.8.0_121-32bit.tar.gz
```

For basic usage, you can leave the key `app` as "app", or you can change it to
indicate the type of application, such as 'java'.  If you do, then you also must
change `multiapp_app_type` to an array with a matching type.  In the previous
example, it would be `multiapp_app_type: [java]`

The `app` key has meaning in two ways.  For simple use cases, it will become a
subdirectory in the installation path, see [Defaults] for more information.
For more advanced use cases, see
[Using across different groups in one playbook].


## Defaults

The most relevant and useful default variables are below:

`multiapp_app_owner_default` and `multiapp_app_group_default`
are both `ansible`.  Note that in order to set the owner and group, they
both must exist.  This role will not attempt to create them, due to the
multiple ways users and groups might be managed.  Add whatever is appropriate
to your playbooks before this role is included (or in `pre_tasks` for older
style).

`multiapp_app_install_dir_mode_default` and `multiapp_app_archive_mode_default`
are both "0755"

`multiapp_app_install_dir_default` is `/apps`, however the default behavior
will add an additional subdirectory to this.  If you change `multiapp_app_type`
to `[java]` and one of your top level `multiapp_app_installtions` keys is
`java`, you will end up with the installations in `/apps/java`.

`multiapp_app_download_path_default` is /tmp/multiapp

`multiapp_app_remove_download_default` is false.

`multiapp_download_timeout_default` is 60 (in seconds)

`multiapp_archive_extension_default` is `tar.gz` see [Archive Format] for more
info.


Optionally, you can override many defaults per installation,
by removing `multiapp_`  and `_default` from any variable you see in
defaults/main.yml.  For example, to set the app owner differently for a single
app installation (provided the Ansible user has sufficient priviliges for doing
so):

```
multiapp_app_installations:
  - app_version: 1.8.0_121-32bit
    app_extracted_dir: jdk1.8.0_121-32bit
    app_artifact: http://some.internal.repo/mirrors/jdk1.8.0_121-32bit.tar.gz
    app_owner: foo
  - app_version: 1.8.0_121-64bit
    app_extracted_dir: jdk1.8.0_121-64bit
    app_artifact: http://some.internal.repo/mirrors/jdk1.8.0_121-64bit.tar.gz
```
See other examples in the [Linking] and the [Archive Format] sections


## Become / sudo

You can indicate that the Ansible user should *not* attempt a become by setting
`multiapp_become` to false.


## Linking

You can symlink any installation by adding `app_symlink` to the hash.  For
example, to link a JDK installations as `latest`:

```
- app_version: 1.8.0_121-64bit
  app_extracted_dir: jdk1.8.0_121-64bit
  app_artifact: http://some.internal.repo/mirrors/jdk1.8.0_121-64bit.tar.gz
  app_symlink: latest
```

would result in `/apps/java/jdk1.8.0_121-64bit` and `/apps/java/latest` being
symlinked to `/apps/java/jdk1.8.0_121-64bit`


## Archive Format

The default archive format is tarball with extension "tar.gz", but overriding is
simple.  Indicate the extension in the installation hash:

```
- app_version: v6.10.2-linux-x64
  app_extracted_dir: node-v6.10.2-linux-x64
  app_artifact: https://nodejs.org/dist/v6.10.2/node-v6.10.2-linux-x64.tar.xz
  app_archive_extension: tar.xz
  app_symlink: latest
```


## Overriding shortcut

To save time on future runs, a shortcut is enabled by default.  This may
present problems if you'd like to re-run a playbook but make changes to the
existing installations, for example, to update the user or group.  To do so,
you can set the `multiapp_ignore_tag_file` to true.


## Using across different groups in one playbook

In order to use across multiple groups in a combined playbook,
you must set the following variables at the **host level**:

in `host_vars/abc.yml`:

```
multiapp_app_type: [java, tomcat]
```

and then include the same top level keys in the installation hash:

in `group_vars/servers.yml`, which server `abc` is a member of:

```
- multiapp_app_installations:
  java:
    - app_version: 1.8.0_112-32bit
      app_extracted_dir: jdk1.8.0_112-32bit
      app_artifact: http://some.internal.repo/mirrors/jdk1.8.0_112-32bit.tar.gz
  tomcat:
    - app_version: 7.0.75
      app_extracted_dir: apache-tomcat-7.0.75
      app_artifact: http://some.internal.repo/mirrors/apache-tomcat-7.0.75.tar.gz
```

You MUST run Ansible with the hash_behavior=merge configured for this to work.
The easiest way to do this is to set the environment variable:

`ANSIBLE_HASH_BEHAVIOUR=merge ansible-playbook /ansible-playbooks/whatevs.yml`


## Purging unwanted

To purge unwanted installations, you can add a param `app_state`` to the
insallation hash.

```

- multiapp_app_installations:
  java:
    - app_version: 1.8.0_121-64bit
      app_extracted_dir: jdk1.8.0_121-64bit
      app_artifact: http://some.internal.repo/mirrors/jdk1.8.0_121-64bit.tar.gz
      app_symlink: latest
    - app_extracted_dir: jdk1.8.0_112-32bit
      app_state: absent
```
It will respected custom install paths, if passed, however, recall how that
works.  In the example above, adding `app_install_dir: /var/apps` would purge
the installation at `/var/apps/java/jdk1.8.0_112-32bit`

Requirements
------------

If you plan on using this role across different inventory groups and mixing
installations (a somehwat unusual, but possible setup), then you must
run Ansible with the hash_behavior=merge configuration.  Please see
"Using across different groups in one playbook" in this README


Role Variables
--------------

There are many more possible.  Please read the rest of this README for more
information.

multiapp_app_type: sets the type of the installation.  Not required for simple
usage.  Default "app"

multiapp_app_owner_default: Not required.  Default "ansible"
multiapp_app_group_default: Not required.  Default "ansible"

multiapp_app_install_dir_mode_default: Not required.  Default "0755"
multiapp_app_archive_mode_default: Not require. Default "0755"

multiapp_app_install_dir_default: Not required.  Default "/apps"
multiapp_app_download_path_default: Not required.  Default "/tmp/multiapp"

multiapp_app_remove_download_default: Not required. Default false
multiapp_download_timeout_default: Not required.  Default 60
multiapp_archive_extension_default: Not required.  Default "tar.gz"

multiapp_become: Not required.  Defaults to true

multiapp_ignore_tag_file: Not required.  Defaults to false

multiapp_debug_output: Not required.  Default false


Dependencies
------------

N/A

Example Playbooks
----------------


#### Basic

```
- hosts: all
  tasks:
    - include_role:
      name: rubyisbeautiful.multi-app
```

License
-------

MIT


Author Information
------------------

rubyisbeautiful

Thanks to some helpful suggestions from my colleagues