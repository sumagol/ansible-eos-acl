ACL Role for EOS
================

The arista.eos-acl role creates an abstraction for common EOS ACL configuration.
This means that you do not need to write any ansible tasks. Simply create an
object that matches the requirements below and this role will ingest that
object and perform the necessary configuration.

This role is used to configure a limited set of ACL types and configurations.
See notes under [Role Variables](#Role Variables) for clarification.

Installation
------------

```
ansible-galaxy install arista.eos-acl
```


Requirements
------------

Requires an SSH connection for connectivity to your Arista device. You can use
any of the built-in eos connection variables, or the convenience ``provider``
dictionary.

Role Variables
--------------

The tasks in this role are driven by the ``acls`` object described below:


**acls** (list) each entry contains the following keys:

|          Key | Type                                 | Notes                                    |
| -----------: | ------------------------------------ | ---------------------------------------- |
|         name | string (required)                    | The name of the ACL to manage.           |
|         type | choices: standard*                   | The type of ACL to manage. Currently the only supported value for acltype is 'standard'. |
|        seqno | int (required if state is 'present') | The sequence number of the ACL rule entry to be used. Will be ignored when state is set to 'absent'. |
|       action | string (required)                    | The action associated with the ACL rule. Currently supports 'permit' and 'deny'. |
|      srcaddr | string (required)                    | The source address filter string for the ACL rule, in the form A.B.C.D. |
| srcprefixlen | choices: 1-32*                       | The prefix length applied to srcaddr. If not 32, the CIDR address specified by srcaddr/srcprefixlen must be a valid network CIDR. Default is 32, using srcaddr as a specific host IP address. |
|          log | boolean: true, false*                | Enables or disables log messages for this ACL rule. |
|        state | choices: present*, absent            | Set the state for the ACL rule configuration. |



```
Note: Asterisk (*) denotes the default value if none specified
```
```
Note: The address specified by srcaddr/srcprefixlen must be either a network
address in CIDR notation, or a single IP address (with srcprefixlen set or
defaulted to 32). An ACL rule specification that does not meet this requirement
will be skipped in the playbook execution.
```
```
Note: Existing ACL entries on the switch that match the action/network/log
configuration of a given ACL specification will be removed from the switch
by the eos-acl role without regard for the specified sequence number. When
state is 'present', all other matching rules will be removed. When state is
'absent', all matching rules will be removed. See example playbook below.
```

Dependencies
------------

The eos-acl role is built on modules included in the core Ansible
code. These modules were added in ansible version 2.1.

- Ansible 2.1.0

Example Playbook
----------------

The following example will use the arista.eos-acl role to configure ACLs on
a switch. We'll create a ``hosts`` file with our switch, then a corresponding
``host_vars`` file and then a simple playbook which only references the
eos-acl role. By including the role, we automatically get access to all of
the tasks to configure these EOS features. What's nice about this is that
if you have a host without any corresponding configuration, the tasks will
be skipped without any issue.


Sample hosts file:

    [leafs]
    leaf1.example.com

Sample host_vars/leaf1.example.com

    acls:
      - name: ACL-1
        type: standard
        # Add '50 permit host 10.10.10.10' to ACL-1, replacing the
        # existing rule number 50 if it exists, and removing any other
        # rules that match '<##> permit host 10.10.10.10' regardless
        # of sequence number.
        seqno: 50
        action: permit
        srcaddr: 10.10.10.10
        srcprefixlen: 32
      - name: ACL-1
        type: standard
        # Add '55 deny 20.20.0.0/16 log' to ACL-1, replacing the
        # existing rule number 55 if it exists, and removing any other
        # rules that match '<##> deny 20.20.0.0 log' regardless of
        # sequence number.
        seqno: 55
        action: deny
        srcaddr: 20.20.0.0
        srcprefixlen: 16
        log: true
      - name: ACL-1
        type: standard
        # Remove all ACL-1 rules that match '<##> permit 30.30.30.0/24 log'
        # regardless of their sequence number.
        action: permit
        srcaddr: 30.30.30.0
        srcprefixlen: 24
        log: true
        state: absent

A simple playbook, leaf.yml

    - hosts: leafs
      roles:
        - arista.eos-acl

Then run with:

    ansible-playbook -i hosts leaf.yml

License
-------

Copyright (c) 2015, Arista Networks EOS+
All rights reserved.

Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions are met:

* Redistributions of source code must retain the above copyright notice, this
  list of conditions and the following disclaimer.

* Redistributions in binary form must reproduce the above copyright notice,
  this list of conditions and the following disclaimer in the documentation
  and/or other materials provided with the distribution.

* Neither the name of Arista nor the names of its
  contributors may be used to endorse or promote products derived from
  this software without specific prior written permission.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

Author Information
------------------

Please raise any issues using our GitHub repo or email us at ansible-dev@arista.com

[quickstart]: http://ansible-eos.readthedocs.org/en/latest/quickstart.html