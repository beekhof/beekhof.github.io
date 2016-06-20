---
date: 2016-06-20 12:45
title: Working with OpenStack Images
excerpt: Some tips for creating and extending OpenStack images
tags:
- heat
- openstack
---

## Creating Images

For creating images, I recommend the `virt-builder` tool that ships
with RHEL based distributions and possibly others:

```
virt-builder centos-7.2 --format qcow2 --install "cloud-init" --selinux-relabel
```

Note the use of the `--selinux-relabel` option.  If you specify
`--install` but do not include this option, you may end up with
instances that treats all attempts to log in as security violations
and blocks them.

The `cloud-init` package is incredibly useful (discussed later) but
isn't available in CentOS images by default, so I recommend adding it
to any image you create.

For the full list of supported targets, try `virt-builder -l`.
Targets should include CirrOS as well as several versions of openSUSE,
Fedora, CentOS, Debian, and Ubuntu.

## Adding Packages to an existing Image

On RHEL based distributions, the `virt-customize` tool is available
and makes adding a new package to an existing image simple.

```
virt-customize  -v -a myImage --install "wget,ntp" --selinux-relabel
```

Note once again the use of the `--selinux-relabel` option.  This
should only be used for the last step of your customization.  As
above, not doing so may result in an instance that treats all attempts
to log in as security violations and blocks them.

Richard Jones also has a good post about [updating RHEL
images](https://rwmj.wordpress.com/2015/10/03/tip-updating-rhel-7-1-cloud-images-using-virt-customize-and-subscription-manager/)
since they require subscriptions. Just be sure to use
`--sm-unregister` and `--selinux-relabel` at the very end.

## Logging in

If you haven't already, tell OpenStack about your keypair:

```
nova keypair-add myKey --pub-key ~/.ssh/id_rsa.pub
```

Now you can tell your provisioning tool to add it to the instances it
creates.  For `Heat`, the template would look like this:

```
myInstance:
  type: OS::Nova::Server
  properties:
    image: { get_param: image }
    flavor: { get_param: flavor }
    key_name: myKey
```

However almost no image will let you log in, via ssh or on the console, as
root.  Instead they normally create a new user that has full `sudo`
access.  Red Hat images default to `cloud-user` while CentOS has a
`centos` user.

If you don't already know which user your instance has, you can use
`nova console-log myImage` to see what happens at boot time.

Assuming you configured a key to add to the instance, you might see a
line such as:

```
ci-info: ++++++Authorized keys from /home/cloud-user/.ssh/authorized_keys for user cloud-user+++++++
```

which tells you which user your image supports.


## Customizing an Instance at Boot Time

This section relies heavily on the
[cloud-init](https://launchpad.net/cloud-init) package.  If it is not
present in your images, be sure to add it using the techniques above
before trying anything below.

### Running Scripts

Running scripts on the instances once they're up can be a useful way
to customize your images, start services and generally work-around
bugs in officially provided images.

The list of commands to run is specified as part of the `user_data`
section of a Heat template or can be passed to `nova boot` with the
`--user-data` option:

```
myNode:
  type: OS::Nova::Server
  properties:
    image: { get_param: image }
    flavor: { get_param: flavor }
    user_data_format: RAW
    user_data:
      #!/bin/sh -ex

      # Fix broken qemu/strstr()
      # https://bugzilla.redhat.com/show_bug.cgi?id=1269529#c9
      touch /etc/sysconfig/64bit_strstr_via_64bit_strstr_sse2_unaligned
```

Note the extra options passed to `/bin/sh` The `e` tells the script to
terminate if any command produces an error and the `x` tells the shell
to log everything that is being executed.  This is particularly useful
as it causes the script's execution to be available in the console's
log (`nova console-log myServer`).

### When Scripts Take a Really Long Time

If we have scripts that take a really long time, we may want to delay
the creation of subsequent resources until our instance is fully
configured.

If we are using Heat, we can set this up by creating SwiftSignal and
SwiftSignalHandle resources to coordinate resource creation with
notifications/signals that could be coming from sources external or
internal to the stack.

```
signal_handle:
  type: OS::Heat::SwiftSignalHandle

wait_on_server:
  type: OS::Heat::SwiftSignal
  properties:
    handle: {get_resource: signal_handle}
    count: 1
    timeout: 2000
```

We then add a layer of indirection to the `user_data:` portion of the
instance definition using the `str_replace:` function to replace all
occurences of "wc_notify" in the script with an appropriate curl PUT
request using the "curl_cli" attribute of the SwiftSignalHandle
resource.

```
myNode:
  type: OS::Nova::Server
  properties:
    image: { get_param: image }
    flavor: { get_param: flavor }
    user_data_format: RAW
    user_data:
      str_replace:
        params:
          wc_notify:   { get_attr: ['signal_handle', 'curl_cli'] }
        template: |
          #!/bin/sh -ex

          my_command_that --takes-a-really long-time

          wc_notify --data-binary '{"status": "SUCCESS", "data": "Script execution succeeded"}'
```

Now the creation of `myNode` will only be considered successful if and
when the script completes.

### Installing Packages

One should avoid the temptation to hardcode calls to a specific
package manager as part of a script as it limits the usefulness of
your template.  Instead, this is done in a platform agnostic way using
the packages directive.

Note that instance creation will not fail if packages fail to install
or are already present.  Check for any required binaries or files as
part of the script.

```
user_data_format: RAW
user_data:
  #cloud-config
  # See http://cloudinit.readthedocs.io/en/latest/topics/examples.html
  packages:
    - ntp
    - wget
```

Note that this will NOT work for images that need a Red Hat
subscription.  There is supposed to be a way to have it register,
however I've had [no
success](https://bugzilla.redhat.com/show_bug.cgi?id=1340323) with
this method and instead I recommend you create a new image that has
any packages listed here pre-installed.

### Installing Packages _and_ Running scripts

The first line of the `user_data:` section (`#config` or `#!/bin/sh`)
is used to determine how it should be interpreted. So if we wish to
take advantage of scripting and `cloud-init`, we must combine the two
pieces into a multi-part MIME message.

The `cloud-init` docs include a [MIME helper
script](http://cloudinit.readthedocs.io/en/latest/topics/format.html#helper-script-to-generate-mime-messages)
to assist in the creation of complex `user_data:` blocks.

Simply create a file for each section and invoke with a command line similar to:

```
python ./mime.py cloud.config:text/cloud-config cloud.sh:text/x-shellscript
```

The resulting output can then be pasted in as a template and even
edited in-place later.  Here is an example that includes notification
for a long running process:

```
user_data_format: RAW
user_data:
  str_replace:
    params:
      wc_notify:   { get_attr: ['signal_handle', 'curl_cli'] }
    template: |
      Content-Type: multipart/mixed; boundary="===============3343034662225461311=="
      MIME-Version: 1.0
      
      --===============3343034662225461311==
      MIME-Version: 1.0
      Content-Type: text/cloud-config; charset="us-ascii"
      Content-Transfer-Encoding: 7bit
      Content-Disposition: attachment; filename="cloud.config"

      #cloud-config
      packages:
        - ntp
        - wget

      --===============3343034662225461311==
      MIME-Version: 1.0
      Content-Type: text/x-shellscript; charset="us-ascii"
      Content-Transfer-Encoding: 7bit
      Content-Disposition: attachment; filename="cloud.sh"
      
      #!/bin/sh -ex

      my_command_that --takes-a-really long-time

      wc_notify --data-binary '{"status": "SUCCESS", "data": "Script execution succeeded"}'
```

