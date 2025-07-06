# Syntax reference for the tailnet policy file

You can write Tailscale [access control](https://tailscale.com/kb/1393/access-control) rules such as [ACLs](https://tailscale.com/kb/1018/acls) and [grants](https://tailscale.com/kb/1324/grants) in the tailnet policy file, which is expressed in [human JSON (HuJSON)](https://github.com/tailscale/hujson).

The tailnet policy file has the following top-level sections:

| Section | Key | Type | Purpose |
| --- | --- | --- | --- |
| [Grants](https://tailscale.com/kb/1337/policy-syntax#grants) | `grants` | Access control | Create network-level and application-level access control policies with optional route filtering. <br>Prefer grants for access control policies. |
| [ACLs](https://tailscale.com/kb/1337/policy-syntax#acls) | `acls` | Access control | Create network-level access control policies. |
| [SSH](https://tailscale.com/kb/1337/policy-syntax#ssh) | `ssh` | Access control | Specify who can use Tailscale SSH. |
| [Auto approvers](https://tailscale.com/kb/1337/policy-syntax#autoapprovers) | `autoApprovers` | Automation | Specify who can bypass the approval process to advertise subnet routers, exit nodes, and app connectors. |
| [Node attributes](https://tailscale.com/kb/1337/policy-syntax#nodeattrs) | `nodeAttrs` | Attributes | Apply additional attributes to devices and users. |
| [Postures](https://tailscale.com/kb/1337/policy-syntax#postures) | `postures` | Attributes | Define device posture rules to target in access control policies. |
| [Tag owners](https://tailscale.com/kb/1337/policy-syntax#tag-owners) | `tagOwners` | Targets | Define who can assign which tags to devices in your tailnet. |
| [Groups](https://tailscale.com/kb/1337/policy-syntax#groups) | `groups` | Targets | Define named groups of users, devices, and subnets to target in access control policies and other definitions. |
| [Hosts](https://tailscale.com/kb/1337/policy-syntax#hosts) | `hosts` | Targets | Define named aliases for devices and subnets. |
| [IP sets](https://tailscale.com/kb/1337/policy-syntax#ipsets) | `ipsets` | Targets | Define named network segments to target in access control policies and other definitions. |
| [Tests](https://tailscale.com/kb/1337/policy-syntax#tests) | `tests` | Tests | Write tests to make assertions about access policies (ACLs and network-level grants) that should not change. |
| [SSH test](https://tailscale.com/kb/1337/policy-syntax#sshtests) | `sshTests` | Tests | Write tests to make assertions about Tailscale SSH that should not change. |

## [Grants](https://tailscale.com/kb/1337/policy-syntax\#grants)

Grants are a new, more powerful approach to access control. They let you do everything you can with ACLs, plus more. When communicating with a destination device, you can grant [application layer](https://en.wikipedia.org/wiki/Application_layer) capabilities to a set of devices or users. You can also continue to define traditional [network layer](https://en.wikipedia.org/wiki/Network_layer) capabilities. For example, you can use a grant rule to give a group of users access to port `8443` on a server, _and_ define the files they can edit on that server.

The grants system combines network layer and application layer capabilities into a shared syntax. As a result, it offers enhanced flexibility and fine-grained control over resource access. Each grant only requires a source and a destination. Because Tailscale takes a deny-by-default approach, each grant has an implied _accept_ action.

[**Get started with grants** \\
\\
Grant access control permissions across both network connections and application permissions.](https://tailscale.com/kb/1324/grants)

## [ACLs](https://tailscale.com/kb/1337/policy-syntax\#acls)

Tailscale now secures access to resources using [grants](https://tailscale.com/kb/1324/grants), a next-generation access control policy syntax. Grants provide [all original ACL functionality plus additional capabilities](https://tailscale.com/kb/1467/grants-vs-acls).

ACLs will continue to work **indefinitely**; Tailscale will not remove support for this first-generation syntax from the product. However, Tailscale recommends [migrating to grants](https://tailscale.com/kb/1542/grants-migration) and using grants for all new tailnet policy file configurations because ACLs will not receive any new features.

The `acls` section lists access rules for your tailnet. Each rule grants access from a set of sources to a set of destinations.

Access rules can use [groups](https://tailscale.com/kb/1337/policy-syntax#groups) and [tags](https://tailscale.com/kb/1068/tags) to grant access to pre-defined sets of users and assign service role accounts to nodes. Together, groups and tags let you build powerful [role-based access control (RBAC)](https://tailscale.com/blog/rbac-like-it-was-meant-to-be) policies.

Tailscale automatically translates all ACLs to lower-level rules that allow traffic from a source IP address to a destination IP address and port.

The following example shows an access rule with an `action`, `src`, `proto`, and `dst`.

```json
{
  "action": "accept",
  "src": [ <list-of-sources> ],
  "proto": "tcp", // optional
  "dst": [ <list-of-destinations> ],
}

```

The `acl` section of the tailnet policy supports the legacy fields `users` and `ports`, but the best practice is to use `src` (instead of `users`) and `dst` (instead of `ports`).

### [`action`](https://tailscale.com/kb/1337/policy-syntax\#action)

Tailscale access rules deny access by default. As a result, the only possible `action` is `accept`. `accept` allows traffic from the source ( `src`) to the destination ( `dst`).

### [`src`](https://tailscale.com/kb/1337/policy-syntax\#src)

The `src` field specifies a list of sources to which the rule applies. Each element in the list can be one of the following:

| **Type** | **Example** | **Description** |
| --- | --- | --- |
| Any | `*` | All traffic originating from Tailscale devices in your tailnet, any approved subnets and `autogroup:shared`. It does not allow traffic originating from non-tailscale devices (unless it is an approved route). |
| User | `shreya@example.com` | Includes all the provided user's devices. |
| [Group](https://tailscale.com/kb/1337/policy-syntax#groups) | `group:<group-name>` | Includes all users in the provided group. |
| Tailscale IP | `100.101.102.103` | Includes only the device that owns the provided Tailscale IP. IPv6 addresses must follow the format `[1:2:3::4]:80`. |
| [Subnet](https://tailscale.com/kb/1019/subnets) CIDR Range | `192.168.1.0/24` | Includes any IP address within the provided subnet. |
| [Host](https://tailscale.com/kb/1337/policy-syntax#hosts) | `my-host` | Includes the Tailscale IP address or CIDR in the `hosts` section. |
| [Tag](https://tailscale.com/kb/1068/tags) | `tag:production` | Includes all devices with the provided tag. |
| [Autogroup](https://tailscale.com/kb/1337/policy-syntax#autogroups) | `autogroup:<role|property>` | Includes devices of users, destinations, or usernames with the same properties or roles. |
| [Autogroup (all)](https://tailscale.com/kb/1337/policy-syntax#autogroups) | `autogroup:danger-all` | A special autogroup that selects all sources including those outside your tailnet. |

You can optionally include the `srcPosture` field to further restrict `src` devices to the ones matching a set of [device posture conditions](https://tailscale.com/kb/1288/device-posture#device-posture-conditions).

### [`proto`](https://tailscale.com/kb/1337/policy-syntax\#proto)

The `proto` field is an optional field you can use to specify the protocol to which the rule applies. Without a protocol, the access rule applies to all TCP and UDP traffic.

You can specify `proto` as an [IANA IP protocol number](https://www.iana.org/assignments/protocol-numbers/protocol-numbers.xhtml) `1-255` (for example, `"16"`) or one of the supported named aliases.

Expand to view all named aliases.

| **Protocol** | **`proto`** | **IANA protocol number** |
| --- | --- | --- |
| Internet Group Management (IGMP) | `igmp` | `2` |
| IPv4 encapsulation | `ipv4`, `ip-in-ip` | `4` |
| Transmission Control (TCP) | `tcp` | `6` |
| Exterior Gateway Protocol (EGP) | `egp` | `8` |
| Any private interior gateway | `igp` | `9` |
| User Datagram (UDP) | `udp` | `17` |
| Generic Routing Encapsulation (GRE) | `gre` | `47` |
| Encap Security Payload (ESP) | `esp` | `50` |
| Authentication Header (AH) | `ah` | `51` |
| Stream Control Transmission Protocol (SCTP) | `sctp` | `132` |

Notes about the `proto` field:

- You must use Tailscale version v1.18.2 or later to use the `proto` field. Earlier versions of Tailscale will fail and block access rules with protocols.
- If traffic is allowed for a given pair of IP addresses, then ICMP will also be allowed.
- Only TCP, UDP, and SCTP traffic support specifying ports. All other protocols only support `*` as the protocol port.

### [`dst`](https://tailscale.com/kb/1337/policy-syntax\#dst)

The `dst` field specifies a list of destinations to which the rule applies. Each element in the list specifies a `host` and one or more `ports` in the format `<host>:<ports>`.

The `host` can be any of the following types:

| **Type** | **Example** | **Description** |
| --- | --- | --- |
| Any | `*` | Includes any destination (no restrictions). |
| User | `shreya@example.com` | Includes any device currently signed in as the provided user. |
| [Group](https://tailscale.com/kb/1337/policy-syntax#groups) | `group:<group-name>` | Includes all users in the provided group. |
| Tailscale IP address | `100.101.102.103` | Includes only the device that owns the provided Tailscale IP address. |
| [Hosts](https://tailscale.com/kb/1337/policy-syntax#hosts) | `example-host-name` | Includes the Tailscale IP address in the [`hosts` section](https://tailscale.com/kb/1337/policy-syntax#hosts). |
| [Subnet](https://tailscale.com/kb/1019/subnets) CIDR Range | `192.168.1.0/24` | Includes any IP address within the given subnet. |
| [Tags](https://tailscale.com/kb/1068/tags) | `tag:<tag-name>` | Includes any device with the provided tag. |
| Internet access through an [exit node](https://tailscale.com/kb/1103/exit-nodes) | `autogroup:internet` | Includes devices with access to the internet through [exit nodes](https://tailscale.com/kb/1103/exit-nodes). |
| Own devices | `autogroup:self` | Includes devices where the same user is authenticated on both the `src` and the `dst`. This does not include devices for which the user has [tags](https://tailscale.com/kb/1068/tags). |
| Tailnet devices | `autogroup:member` | Includes devices in the tailnet where the user is a direct member (not a shared user) of the tailnet. |
| Admin devices | `autogroup:admin` | Includes devices where the user is an [Admin](https://tailscale.com/kb/1138/user-roles#admin). |
| Network admin devices | `autogroup:network-admin` | Includes devices where the user is a [Network admin](https://tailscale.com/kb/1138/user-roles#network-admin). |
| IT admin devices | `autogroup:it-admin` | Includes to devices where the user is an [IT admin](https://tailscale.com/kb/1138/user-roles#it-admin). |
| Billing admin devices | `autogroup:billing-admin` | Includes devices where the user is a [Billing admin](https://tailscale.com/kb/1138/user-roles#billing-admin). |
| Auditor devices | `autogroup:auditor` | Includes devices where the user is an [Auditor](https://tailscale.com/kb/1138/user-roles#auditor). |
| Owner devices | `autogroup:owner` | Includes devices where the user is the tailnet [Owner](https://tailscale.com/kb/1138/user-roles#owner). |
| [IP sets](https://tailscale.com/kb/1387/ipsets) | `ipset:<ip-set-name>` | Includes all targets in the IP set. |

The `ports` field can be any of the following types:

| **Type** | **Description** | **Example** |
| --- | --- | --- |
| Any | Includes any port number. | `*` |
| Single | Includes a single port number. | `22` |
| Multiple | Includes two or more port numbers separated by commas. | `80,443` |
| Range | Includes a range of port numbers. | `1000-2000` |

### [Subnet routers and exit nodes](https://tailscale.com/kb/1337/policy-syntax\#subnet-routers-and-exit-nodes)

ACLs don't limit the discovery of routes. If a device is a [subnet router](https://tailscale.com/kb/1019/subnets), you can restrict access to it independently from the subnet. If a device is an [exit node](https://tailscale.com/kb/1103/exit-nodes), you can restrict access to it independently from its public IP address.

To restrict access to a subnet, ensure that no ACL allows access to those routes. You can enforce this with a test that fails if any rule accidentally allows access. The following example demonstrates a test that fails if `not-allowed@example.com` is allowed access to `198.51.100.7:22`.

```json
"tests": [\
   {\
     "src": "not-allowed@example.com",\
     "accept": ["192.0.2.100:22"], // allow access to the tailscale IP\
     "deny": ["198.51.100.7:22"], // does not allow access to the subnet\
   }\
],

```

Only devices with access to `autogroup:internet` can use exit nodes. All other devices (without access to `autogroup:internet`) cannot use exit nodes. You can enforce this with a test that fails if any rule accidentally allows access to a public address. The following example test fails if `not-allowed@example.com` can access `198.51.100.8:22`.

```json
"tests": [\
   {\
     "src": "not-allowed@example.com",\
     "accept": ["192.0.2.100:22"], // allow access to the tailscale IP\
     "deny": ["198.51.100.8:22"], // does not allow access to a public IP\
   }\
],

```

You cannot restrict the use of specific exit nodes using ACLs. Refer to [issue #1567](https://github.com/tailscale/tailscale/issues/1567) for updates.

### [Taildrop precedence](https://tailscale.com/kb/1337/policy-syntax\#taildrop-precedence)

Taildrop permits you to share files between devices you're logged in to, even if you use ACLs to restrict access.

## [Reference users](https://tailscale.com/kb/1337/policy-syntax\#reference-users)

Usersare available for [the Personal, Personal Plus, Premium, and Enterprise plans](https://tailscale.com/pricing).

You can specify users in an access rule's source ( `src`) and destination ( `dst`) fields. To specify a user, use one of the following formats (depending on how the user signs into Tailscale):

| **Format** | **Description** | **Example** |
| --- | --- | --- |
| `username@example.com` | Use if the user signs into Tailscale with an email address. | `alice@example.com` |
| `username@github` | Use if the user signs into Tailscale with a GitHub account. | `alice@github` |
| `username@passkey` | Use if the user signs into Tailscale with a Passkey. | `alice@passkey` |

You can use groups to reference sets of users. Groups let you define role-based access controls. There are multiple types of groups:

- Auto groups that reference all users with the same property.
- Groups defined in the `groups` section of the tailnet policy file as a specific list of users.
- Groups provisioned in the identity provider and synced through user and group provisioning.

## [Autogroups](https://tailscale.com/kb/1337/policy-syntax\#autogroups)

Autogroupsare available for [all plans](https://tailscale.com/pricing).

An [autogroup](https://tailscale.com/kb/1396/targets#autogroups) is a special group that automatically includes users, destinations, or usernames with the same properties.

| **Allowed** | **Autogroup** | **Description** | **Availability by plan** |
| --- | --- | --- | --- |
| As a `dst` | `autogroup:internet` | Use to allow access for any user through _any_ [exit node](https://tailscale.com/kb/1103/exit-nodes) in your tailnet. | Available on [all plans](https://tailscale.com/pricing) |
| As a `dst` | `autogroup:self` | Use to allow access for any user that is authenticated as the same user as the source. Does not apply to tags. | Available on [all plans](https://tailscale.com/pricing) |
| As a `src` or `dst`, `tagOwner`, or `autoApprover` | `autogroup:owner` | Use to allow access for the tailnet [Owner](https://tailscale.com/kb/1138/user-roles#owner). | Available on [all plans](https://tailscale.com/pricing) |
| As a `src` or `dst`, `tagOwner`, or `autoApprover` | `autogroup:admin` | Use to allow access for any user who has the role of [Admin](https://tailscale.com/kb/1138/user-roles#admin). | Available on [all plans](https://tailscale.com/pricing) |
| As a `src` or `dst`, `tagOwner`, or `autoApprover` | `autogroup:member` | Use to allow access for any user who is a direct member (including all invited users) of the tailnet. Does not include users from shared devices. | Available on [all plans](https://tailscale.com/pricing) |
| As a `src` or `dst`, `tagOwner`, or `autoApprover` | `autogroup:tagged` | Use to allow access for any user who is a device that is [tagged](https://tailscale.com/kb/1068/tags). | Available on [all plans](https://tailscale.com/pricing) |
| As a `src` or `dst`, `tagOwner`, or `autoApprover` | `autogroup:auditor` | Use to allow access for any user who has the role of [Auditor](https://tailscale.com/kb/1138/user-roles#auditor). | Available on [the Personal, Personal Plus, Premium, and Enterprise plans](https://tailscale.com/pricing) |
| As a `src` or `dst`, `tagOwner`, or `autoApprover` | `autogroup:billing-admin` | Use to allow access for any user who has the role of [Billing admin](https://tailscale.com/kb/1138/user-roles#billing-admin). | Available on [the Personal, Personal Plus, Premium, and Enterprise plans](https://tailscale.com/pricing) |
| As a `src` or `dst`, `tagOwner`, or `autoApprover` | `autogroup:it-admin` | Use to allow access for any user who has the role of [IT admin](https://tailscale.com/kb/1138/user-roles#it-admin). | Available on [the Personal, Personal Plus, Premium, and Enterprise plans](https://tailscale.com/pricing) |
| As a `src` or `dst`, `tagOwner`, or `autoApprover` | `autogroup:network-admin` | Use to allow access for any user who has the role of [Network admin](https://tailscale.com/kb/1138/user-roles#network-admin). | Available on [the Personal, Personal Plus, Premium, and Enterprise plans](https://tailscale.com/pricing) |
| As a `src` or `dst`, `tagOwner`, or `autoApprover` | `user:*@<domain>` | Use to allow access for any user whose login is in the specified domain and who is a direct member (including all invited users) of the tailnet. Does not include users from shared devices. | Available on [the Starter, Premium, and Enterprise plans](https://tailscale.com/pricing) |
| As a `src` | `autogroup:shared` | Use to allow access for any user who accepted a [sharing](https://tailscale.com/kb/1084/sharing) invitation to your network. This lets you write rules without knowing the email addresses in advance. | Available on [all plans](https://tailscale.com/pricing) |
| As an [SSH](https://tailscale.com/kb/1337/policy-syntax#ssh) user | `autogroup:nonroot` | Use to allow [Tailscale SSH](https://tailscale.com/kb/1193/tailscale-ssh) access to any user that is not `root`. | Available on [the Personal, Personal Plus, Premium, and Enterprise plans](https://tailscale.com/pricing) |
| As an [SSH](https://tailscale.com/kb/1337/policy-syntax#ssh) user | `localpart:*@<domain>` | Use to allow [Tailscale SSH](https://tailscale.com/kb/1193/tailscale-ssh) access to the user whose name matches the [local-part](https://datatracker.ietf.org/doc/html/rfc2822#section-3.4.1) of the user's login. | Available on [the Premium and Enterprise plans](https://tailscale.com/pricing) |

`autogroup:self` only applies to user-owned devices. It does not apply to tagged devices. You cannot use `autogroup:self` with `autogroup:tagged`.

The legacy autogroup `autogroup:members` will continue to work, but it's best practice to use `autogroup:member` instead. You cannot use both `autogroup:member` and `autogroup:members` in the same tailnet policy file.

The following example [`ssh` rule](https://tailscale.com/kb/1337/policy-syntax#ssh) allows all users Tailscale SSH access to devices they own (as non-root):

```json
"ssh": [\
  {\
    // All users can SSH to their own devices, as non-root\
    "action": "accept",\
    "src": ["autogroup:member"],\
    "dst": ["autogroup:self"],\
    "users": ["autogroup:nonroot"]\
  },\
]

```

In the default ACL, the `ssh` rule uses `autogroup:self` for the `dst` field and `autogroup:nonroot` in the `users` field. If you change the `dst` field from `autogroup:self` to some other destination, such as an [ACL tag](https://tailscale.com/kb/1068/acl-tags/), also consider replacing `autogroup:nonroot` in the `users` field. If you don't remove `autogroup:nonroot` from the `users` field, then anyone permitted by the `src` setting will be able to SSH in as any nonroot user on the `dst` device.

### [Domain based autogroups](https://tailscale.com/kb/1337/policy-syntax\#domain-based-autogroups)

Some autogroups include a specific domain name. For example, `user:*@example.com` or `localpart:*@example.com`. These autogroups include users who are both members of the tailnet and whose login is in the autogroup domain. For example, if the tailnet `example.com` uses the autogroup `user:*@altostrat.com`, this group includes all members of the `example.com` tailnet who log in as a user at `@altostrat.com` (such as `laura@altostrat.com`).

The following restrictions apply to the domains used in autogroups:

- The provided domain must not be a known shared domain (such as `gmail.com`).
- If a tailnet uses domain aliases, you must explicitly specify the aliased domains in the ACL. For example, if `example.io` is aliased to `example.com` and you want to include users from both `example.com` and `example.io`, use both `user:*@example.com` and `user:*@example.io`.
- Although the expressions use the wildcard `*`, it does not support arbitrary wildcards. For example, `user:b*b@example.com` will not match `bob@example.com`.

## [Groups](https://tailscale.com/kb/1337/policy-syntax\#groups)

Groupsare available for [the Personal, Personal Plus, Premium, and Enterprise plans](https://tailscale.com/pricing).

The `groups` section lets you create groups of users, which you can use in access rules (instead of listing users out explicitly). Any change you make to the membership of a group propagates to all the rules that reference that group.

The following example demonstrates creating an `engineering` group and a `sales` group.

```json
"groups": {
  "group:engineering": [\
    "dave@example.com",\
    "laura@example.com",\
  ],
  "group:sales": [\
    "brad@example.com",\
    "alice@example.com",\
  ],
},

```

Every group name must start with the prefix `group:`. Each group member is specified by their full email address, as explained in the [users section](https://tailscale.com/kb/1337/policy-syntax#reference-users) above. To avoid the risk of obfuscating group membership, groups cannot contain other groups.

You can add or remove a user's group membership by editing the tailnet policy file, as shown in the example `groups` definition above, and directly from the [**Users**](https://login.tailscale.com/admin/users) page of the admin console.

### [Edit a user's group membership from the Users page](https://tailscale.com/kb/1337/policy-syntax\#edit-a-users-group-membership-from-the-users-page)

You must be an [Owner, Admin, or Network admin](https://tailscale.com/kb/1138/user-roles) to edit a user's group membership from the **Users** page.

1. Open the [**Users**](https://login.tailscale.com/admin/users) page in the admin console.
2. Find the user by name.
3. Select the ![ellipsis icon](https://tailscale.com/files/images/icons/fa-ellipsis-h.svg) menu > **Edit group membership**.
4. In the **Edit group membership** dialog:
1. To add a group, select **Add to a group**, then the group to add.
2. To remove a group, select the **X** next to the group to delete.
5. When you finish editing the groups for the user, select **Save**.

### [Synced groups](https://tailscale.com/kb/1337/policy-syntax\#synced-groups)

You can create groups in your identity provider and sync them with Tailscale's access control policies with [user and group provisioning](https://tailscale.com/kb/1290/user-group-provisioning#syncing-group-membership).

You can use the same human-readable group names in your identity provider to refer to groups in your tailnet policy file. The following example shows an access rule that manages access for the `security-team` group.

```json
{
    "grants": [\
        {\
            "src": ["group:security-team@example.com"],\
            "dst": ["tag:logging"],\
            "ip": ["*"]\
        }\
    ],
    "tagOwners": {
        "tag:logging": ["group:security-team@example.com"]
    }
}

```

You can only edit groups defined in the tailnet policy file. You can use groups synced from a System for Cross-domain Identity Management (SCIM) integration or tailnet autogroups, but you cannot edit them.

## [Reference multiple devices](https://tailscale.com/kb/1337/policy-syntax\#reference-multiple-devices)

You can define access rules for sets of devices using tags or hosts. Tags let you define role-based access controls so that different services have different access rules. Hosts let you define controls based on a reference to an IP address.

- Tags reference groups of non-user devices (such as applications or servers). For example, you might have a tag that groups all servers in a particular data center.
- Hosts reference groups of devices by IP address ranges (both on and beyond the tailnet). For example, you can use hosts to address applications with fixed IP addresses that you might be unable to modify.

### [Tags](https://tailscale.com/kb/1337/policy-syntax\#tags)

Tagsare available for [all plans](https://tailscale.com/pricing).

The `tags` section of the tailnet policy file lets you create [tags](https://tailscale.com/kb/1068/tags) that group non-human devices. You can then use the tags to select these devices in an ACL.

You must [define the tag](https://tailscale.com/kb/1068/tags#define-a-tag) in the [`tagOwners`](https://tailscale.com/kb/1337/policy-syntax#tag-owners) section of the tailnet policy file before using it in an ACL. To tag a device, [authenticate as the tag on the device](https://tailscale.com/kb/1068/tags#apply-a-tag-to-a-device).

### [Hosts](https://tailscale.com/kb/1337/policy-syntax\#hosts)

Hostsare available for [all plans](https://tailscale.com/pricing).

The `hosts` section lets you define a human-friendly name for an IP address or CIDR range.

The following example shows two host definitions: one for a single IP address and one for a CIDR range.

```json
"hosts": {
  "example-host-1": "198.51.100.100",
  "example-network-1": "198.51.100.0/24",
},

```

The human-friendly hostname cannot include the character `@`.

## [Postures](https://tailscale.com/kb/1337/policy-syntax\#postures)

Posturesare available for [all plans](https://tailscale.com/pricing).

The `postures` section lets you define a set of [device posture management](https://tailscale.com/kb/1288/device-posture) rules that a device must meet as part of a specific access rule.

The following example shows how to use `postures` to select macOS devices running `node` version 1.40 or later.

```json
"postures": {
    "posture:latestMac": [\
        "node:os IN ['macos']",\
        "node:tsReleaseTrack == 'stable'",\
        "node:tsVersion >= '1.40'",\
    ],
},

```

Each posture must start with the prefix `posture:` followed by a name, a set of [posture attributes](https://tailscale.com/kb/1288/device-posture#device-posture-attributes), and their allowed values, given as a list of strings.

Refer to [device posture management](https://tailscale.com/kb/1288/device-posture) for more information

## [Tag owners](https://tailscale.com/kb/1337/policy-syntax\#tag-owners)

Tagsare available for [all plans](https://tailscale.com/pricing).

The `tagOwners` section of the tailnet policy file defines the tags assignable to devices and the list of users allowed to assign each tag.

The following example shows a `tagOwners` definition that:

- Sets the `engineering` group as the owner of the `webserver` tag.
- Sets `president@example.com` and the `security-admins` group as owners of the `secure-server` tag.
- Sets the `autogroup:member` autogroup as the owner of the `corp` tag.

```json
"tagOwners": {
  "tag:webserver": [\
    "group:engineering",\
  ],
  "tag:secure-server": [\
    "group:security-admins",\
    "president@example.com",\
  ],
  "tag:corp": [\
    "autogroup:member",\
  ],
}

```

Every tag name must start with the prefix `tag:`. A tag owner can be a user's full login email address (as defined in the [users section](https://tailscale.com/kb/1337/policy-syntax#reference-users) above), a [group name](https://tailscale.com/kb/1337/policy-syntax#groups), an [autogroup](https://tailscale.com/kb/1337/policy-syntax#autogroups), or another tag.

A shorthand notation, `[]`, is available for `autogroup:admin`. That is, the following are equivalent:

```json
"tag:monitoring": [\
  "autogroup:admin",\
],

```

```json
"tag:monitoring": [],

```

The autogroups `autogroup:admin` and `autogroup:network-admin` can assign all tags, so `[]` implicitly allows only `autogroup:admin` and `autogroup:network-admin`.

## [Auto approvers](https://tailscale.com/kb/1337/policy-syntax\#auto-approvers)

Auto approversare available for [all plans](https://tailscale.com/pricing).

The `autoApprovers` section of the tailnet policy file defines the list of users who can perform specific actions without further approval from the admin console. Some actions in Tailscale require double opt-in: an [Admin](https://tailscale.com/kb/1138/user-roles) must enable them on the device running Tailscale and in the Tailscale admin console. These actions include:

- [Advertising a specified set of routes](https://tailscale.com/kb/1019/subnets/connect-to-tailscale-as-a-subnet-router) as a subnet router.
- [Advertising an exit node](https://tailscale.com/kb/1103/exit-nodes#advertise-a-device-as-an-exit-node).

For routes, this also permits the auto approvers to advertise a subnet of the specified routes.

Tailscale stops advertising a route if one of the following occurs:

- The device is re-authenticated by a different user (who cannot advertise the route or exit node).
- The user who advertised the route is suspended or deleted.

To avoid a scenario where Tailscale stops advertising a route, consider using a [tag](https://tailscale.com/kb/1068/tags) as an auto approver.

The following example shows an `autoApprovers` definition that automatically approves the `192.0.2.0/24` routes for `alice@example.com`, members of the `engineering` group, and devices tagged with `foo`. It also automatically allows devices tagged with `foo` to use an exit node.

```json
"autoApprovers": {
  "routes": {
    "192.0.2.0/24": ["group:engineering", "alice@example.com", "tag:foo"],
  },
  "exitNode": ["tag:bar"],
}

```

The auto approver of a route or exit node can be a user's full login email address (as defined in the [users section](https://tailscale.com/kb/1337/policy-syntax#reference-users) above), a [group name](https://tailscale.com/kb/1337/policy-syntax#groups), an [autogroup](https://tailscale.com/kb/1337/policy-syntax#autogroups) or a tag.

## [Tailscale SSH](https://tailscale.com/kb/1337/policy-syntax\#tailscale-ssh)

Tailscale SSHis available for [the Personal, Personal Plus, Premium, and Enterprise plans](https://tailscale.com/pricing).

The `ssh` section of the tailnet policy file defines lists of users and devices that can use [Tailscale SSH](https://tailscale.com/kb/1193/tailscale-ssh) (and the SSH users). To allow a connection, the tailnet policy file must contain rules permitting both network access and SSH access:

1. An access rule to allow connections from the source to the destination on port 22.
2. An SSH access rule to allow connections from the source to the destination and the given SSH users. Tailscale SSH uses this to distribute keys to authenticating SSH connections.

The following example shows an `ssh` definition that requires a list of sources, destinations, and SSH users to re-authenticate every 20 hours.

```json
{
  "action": "check", // "accept" or "check"
  "src": [ <list-of-sources> ],
  "dst": [ <list-of-destinations> ],
  "users": [ <list-of-ssh-users> ],
  "checkPeriod": "20h", // optional, only for check actions. default 12h
  "acceptEnv": [ "GIT_EDITOR", "GIT_COMMITTER_*", "CUSTOM_VAR_V?" ] // optional, allowlists environment variables that can be forwarded from clients to the host
},

```

### [`action`](https://tailscale.com/kb/1337/policy-syntax\#action-1)

Specifies whether to accept the connection or to perform additional checks on it.

- `accept` accepts connections from users already authenticated in the tailnet.
- `check` requires users to periodically reauthenticate according to the `checkPeriod`.

### [`src`](https://tailscale.com/kb/1337/policy-syntax\#src-1)

Specifies the source (where a connection originates from). You can only define an access rule's destination ( `dst`) as yourself, a group, a tag, or an autogroup. You cannot use `*`, other users, IP addresses, or hostnames.

It's impossible to guarantee the ownership of an IP address or hostname when you create an access rule. As a security measure, Tailscale prevents using users, IP addresses, or hostnames in the `dst` field of access rules to protect against scenarios in which one user can unintentionally access a device that doesn't belong to them. Tailscale also prevents any `src` and `dst` combinations that allow multiple users to access a single user's device.

Granting access to `autogroup:members` also allows access to [external invited users](https://tailscale.com/kb/1271/invite-any-user) if the destination device is [shared](https://tailscale.com/kb/1084/sharing) with them, even if they have no devices in your tailnet.

### [`dst`](https://tailscale.com/kb/1337/policy-syntax\#dst-1)

Specifies the destination (where the connection goes). The destination can be a user, tag, or autogroup. You cannot specify a port because only port `22` is allowed. You cannot `*` as the destination.

### [`users`](https://tailscale.com/kb/1337/policy-syntax\#users)

Specifies the set of allowed usernames on the host. Tailscale only uses user accounts that already exist on the host.

- Specify `autogroup:nonroot` to allow any user that is not `root`.
- Specify `localpart:*@<domain>` to allow the user on the host whose name matches the [local-part](https://datatracker.ietf.org/doc/html/rfc2822#section-3.4.1) of the user's login, if and only if the user's login email is in `<domain>`. Tailscale does not do any special processing on the local-part. For example, if the login is `dave+sshuser@example.com`, Tailscale will map this to the ssh user `dave+sshuser`.
- If no user is specified, Tailscale will use the local host's user. That is, if the user is logged in as `alice` locally, then connects with SSH to another device, Tailscale SSH will try to log in as user `alice`.

### [`checkPeriod`](https://tailscale.com/kb/1337/policy-syntax\#checkperiod)

When `action` is `check`, `checkPeriod` specifies the time period for which to allow a connection before requiring a check. You can specify the time in minutes or hours. The time must be at least one minute and at most 168 hours (one week).

- The default check period is 12 hours.
- You can also specify `always` to require a check on every connection. Using `always` might cause unexpected behavior with automation tools that open many SSH connections in quick succession (such as [Ansible](https://ansible.com/)).

### [`acceptEnv`](https://tailscale.com/kb/1337/policy-syntax\#acceptenv)

The host must be running Tailscale v1.76.0 or later to use `acceptEnv`.

Specifies the set of allowlisted environment variable names that clients can send to the host using [`SendEnv`](https://man.openbsd.org/ssh_config#SendEnv) or [`SetEnv`](https://man.openbsd.org/ssh_config#SetEnv).

Values can contain `*` and `?` wildcard characters. `*` matches zero or more characters and `?` matches a single character.

#### [`acceptEnv` examples](https://tailscale.com/kb/1337/policy-syntax\#acceptenv-examples)

| acceptEnv | Permitted | Rejected |
| --- | --- | --- |
| `*` | `FOO_A` `FOO_B` `FOO_OTHER` `BAZ` |  |
| `FOO_*` | `FOO_A` `FOO_B` `FOO_OTHER` | `BAZ` |
| `FOO_?` | `FOO_A` `FOO_B` | `FOO_OTHER` `BAZ` |
| `FOO_A` | `FOO_A` | `FOO_B` `FOO_OTHER` `BAZ` |

### [Order of evaluation](https://tailscale.com/kb/1337/policy-syntax\#order-of-evaluation)

Tailscale evaluates SSH access rules using the most restrictive policies first:

- Check policies
- Accept policies

For example, if you have an access rule allowing the user `alice@example.com` to access a resource with an `accept` rule, and a rule allowing `group:devops` which `alice@example.com` belongs to, to access a resource with a `check` rule, then the `check` rule applies.

Tailnets that have not modified their tailnet policy file have a [default SSH policy](https://tailscale.com/kb/1193/tailscale-ssh#ssh-access-rules-in-default-acl) allowing users to access devices they own using check mode.

The only types of connections that are allowed are:

- From a user to their own devices (as any user, including `root`).
- From a user to a [tagged](https://tailscale.com/kb/1068/tags) device (as any user, including `root`).
- From a tagged device to another tagged device (for any tags). An SSH access rule from a tagged device cannot be in [check mode](https://tailscale.com/kb/1193/tailscale-ssh#configure-tailscale-ssh-with-check-mode).
- From a user to a tagged device that has been [shared](https://tailscale.com/kb/1084/sharing) with them, as long as the destination host has Tailscale configured with SSH and the destination's ACL allows the user to connect over SSH.

That is, the broadest policy allowed would be:

```json
{
    "grants": [\
        {\
            "src": ["*"],\
            "dst": ["*"],\
            "ip": ["*"]\
        }\
    ],
    "ssh": [\
        {\
            "action": "accept",\
            "src": ["autogroup:member"],\
            "dst": ["autogroup:self"],\
            "users": ["root", "autogroup:nonroot"]\
        },\
        {\
            "action": "accept",\
            "src": ["autogroup:member"],\
            "dst": ["tag:prod"],\
            "users": ["root", "autogroup:nonroot"]\
        },\
        {\
            "action": "accept",\
            "src": ["tag:logging"],\
            "dst": ["tag:prod"],\
            "users": ["root", "autogroup:nonroot"]\
        }\
    ]
}

```

To allow a user to only SSH to their own devices (as non- `root`):

```json
{
    "grants": [\
        {\
            "src": ["*"],\
            "dst": ["*"],\
            "ip": ["*"]\
        }\
    ],
    "ssh": [\
        {\
            "action": "accept",\
            "src": ["autogroup:member"],\
            "dst": ["autogroup:self"],\
            "users": ["autogroup:nonroot"]\
        }\
    ]
}

```

To allow `group:sre` to access devices in the production environment tagged `tag:prod`:

```json
{
 "groups": {
   "group:sre": ["alice@example.com", "bob@example.com"]
 },
 "grants": [\
   {\
     "src": ["group:sre"],\
     "dst": ["tag:prod"],\
     "ip": ["*"]\
   }\
 ],
 "ssh": [\
   {\
     "action": "accept",\
     "src": ["group:sre"],\
     "dst": ["tag:prod"],\
     "users": ["ubuntu", "root"]\
   }\
 ],
 "tagOwners": {
   // users in group:sre can apply the tag tag:prod
   "tag:prod": ["group:sre"]
 }
}

```

To allow Alice to access devices in the development environment tagged `tag:dev` that have been [shared](https://tailscale.com/kb/1084/sharing) with them:

```json
{
  "ssh": [\
    {\
      "action": "accept",\
      "src": ["alice@example.com"],\
      "dst": ["tag:dev"],\
      "users": ["root", "alice"]\
    },\
  ]
}

```

It might be useful to match host users with login emails. For example, you can allow `dave@example.com` to authenticate as the host user `dave`.

To allow any tailnet member in the login domain `example.com` to access devices in the production environment that are tagged `tag:prod`, as a user that matches their login email local-part:

```json
{
   "grants": [\
       {\
           "src": ["user:*@example.com"],\
           "dst": ["tag:prod"],\
           "ip": ["*"]\
       }\
   ],
   "ssh": [\
       {\
           "action": "accept",\
           "src": ["user:*@example.com"],\
           "dst": ["tag:prod"],\
           "users": ["localpart:*@example.com"]\
       }\
   ]
}

```

## [Node attributes](https://tailscale.com/kb/1337/policy-syntax\#node-attributes)

The `nodeAttrs` section of the tailnet policy file defines additional attributes that apply to specific devices in your tailnet. You can use node attributes to set different [NextDNS configurations](https://tailscale.com/kb/1218/nextdns) for different devices in your tailnet.

The following example shows a `nodeAttrs` definition that targets `my-kid@my-home.com` and `tag:server` with the attributes `nextdns:abc123` and `nextdns:no-device-info`.

```json
"nodeAttrs": [\
    {\
        "target": ["my-kid@my-home.com", "tag:server"],\
        "attr": [\
            "nextdns:abc123",\
            "nextdns:no-device-info",\
        ],\
    },\
],

```

### [`target`](https://tailscale.com/kb/1337/policy-syntax\#target)

Specifies which nodes (devices) the attributes apply to. You can select the devices using a tag ( `tag:server`), user ( `alice@example.com`), group ( `group:kids`), or `*`.

### [`attr`](https://tailscale.com/kb/1337/policy-syntax\#attr)

Specifies which attributes apply to those nodes (devices).

For example:

- The attribute `nextdns:abc123` specifics the NextDNS configuration ID `abc123`. If this is used, the attribute overrides the global NextDNS configuration.
- The attribute `nextdns:no-device-info` disables sending device metadata to NextDNS.

The following example allows members of the tailnet to use [Tailscale Funnel](https://tailscale.com/kb/1223/funnel) on their nodes:

```json
"nodeAttrs": [\
    {\
         "target": ["autogroup:members"],\
         "attr":   ["funnel"],\
     },\
],

```

## [Tests](https://tailscale.com/kb/1337/policy-syntax\#tests)

Testsare available for [all plans](https://tailscale.com/pricing).

The `tests` section lets you write assertions about your access control policies (grants and ACLs) that run as checks each time the tailnet policy file changes. If an assertion fails, the Tailscale rejects the updated tailnet policy file with an error. The error message indicates the failing tests.

Tests let you ensure you don't accidentally revoke important permissions or expose a critical system.

A `tests` definition looks like this:

```json
"tests": [\
  {\
    "src": "dave@example.com",\
    "srcPostureAttrs": {\
      "node:os": "windows",\
    },\
    "proto": "tcp",\
    "accept": ["example-host-1:22", "vega:80"],\
    "deny": ["192.0.2.3:443"],\
  },\
],

```

### [`src`](https://tailscale.com/kb/1337/policy-syntax\#src-2)

Specifies the user identity to test, which can be a [user's email address](https://tailscale.com/kb/1337/policy-syntax#reference-users), a [group](https://tailscale.com/kb/1337/policy-syntax#groups), a [tag](https://tailscale.com/kb/1068/tags), or a [host](https://tailscale.com/kb/1337/policy-syntax#hosts) that maps to an IP address. The test case runs from the perspective of a device authenticated with the provided identity.

### [`srcPostureAttrs`](https://tailscale.com/kb/1337/policy-syntax\#srcpostureattrs)

Specifies the [device posture attributes](https://tailscale.com/kb/1288/device-posture) as key-value pairs to use when evaluating posture conditions in access rules. You only need to use this field if the access rules contain [device posture conditions](https://tailscale.com/kb/1288/device-posture#device-posture-conditions).

### [`proto`](https://tailscale.com/kb/1337/policy-syntax\#proto-1)

Specifies the IP protocol for `accept` and `deny` rules, similar to the `proto` field in [ACL rules](https://tailscale.com/kb/1337/policy-syntax#acls). When omitted, the test checks for either TCP or UDP access.

### [`accept` and `deny` destinations](https://tailscale.com/kb/1337/policy-syntax\#accept-and-deny-destinations)

Specifies destinations to accept or deny. Each destination in the list is of the form `host:port` where `port` is a single numeric port and `host` is one of the following:

| **Type** | **Example** | **Description** |
| --- | --- | --- |
| Tailscale IP | `100.101.102.103` | Includes the device with the provided Tailscale IP address. IPv6 addresses must follow the format `[1:2:3::4]:80`. |
| [Host](https://tailscale.com/kb/1337/policy-syntax#hosts) | `my-host` | Includes the Tailscale IP address in the `hosts` section. |
| User | `shreya@example.com` | Includes the Tailscale IP addresses of devices signed in as the provided user. |
| [Group](https://tailscale.com/kb/1337/policy-syntax#groups) | `group:security@example.com` | Includes the Tailscale IP addresses of devices signed in as a representative member of the provided group. |
| [Tag](https://tailscale.com/kb/1068/tags) | `tag:production` | Includes the Tailscale IP addresses of devices tagged with the provided tag. |

You cannot use CIDR (subnet) notation to test subnet ranges. For example, `192.168.1.0/24` is not valid. Instead, you must specify the individual IP addresses or hostnames.

Sources in `src` and destinations in `accept` and `deny` must refer to specific entities and do not support `*` wildcards. For example, an `accept` destination cannot be `tags:*`.

The legacy `allow` (instead of `accept`) continues to work in ACLs. However, it is best practice to use `accept`.

## [SSH Tests](https://tailscale.com/kb/1337/policy-syntax\#ssh-tests)

SSH testsare available for [all plans](https://tailscale.com/pricing).

The `sshTests` section lets you write assertions about your [Tailscale SSH](https://tailscale.com/kb/1193/tailscale-ssh) access rules. SSH tests function similarly to ACL [tests](https://tailscale.com/kb/1337/policy-syntax#tests).

SSH tests run when the tailnet policy file changes. If an assertion fails, Tailscale rejects the updated tailnet policy file with an error detailing the failing tests.

The following example shows a `sshTests` definition performs the following tests on connections from `dave@example.com` to `example-host-1`:

- If the user is `dave`, it accepts the connection.
- If the user is `admin`, it checks the connection.
- If the user is `root`, it denies the connection.

```json
"sshTests": [\
  {\
    "src": "dave@example.com",\
    "dst": ["example-host-1"],\
    "accept": ["dave"],\
    "check": ["admin"],\
    "deny": ["root"],\
  },\
],

```

### [`src`](https://tailscale.com/kb/1337/policy-syntax\#src-3)

Specifies the user identity that's attempting to connect as SSH, which can be a [user's email address](https://tailscale.com/kb/1337/policy-syntax#reference-users), a [group](https://tailscale.com/kb/1337/policy-syntax#groups), a [tag](https://tailscale.com/kb/1068/tags), or a [host](https://tailscale.com/kb/1337/policy-syntax#hosts) that maps to an IP address. The test case runs from the perspective of a device authenticated with the provided identity.

### [`dst`](https://tailscale.com/kb/1337/policy-syntax\#dst-2)

Specifies one or more destinations to which the `src` user is connecting, which can be a [user's email address](https://tailscale.com/kb/1337/policy-syntax#reference-users), a [group](https://tailscale.com/kb/1337/policy-syntax#groups), a [tag](https://tailscale.com/kb/1068/tags), or a [host](https://tailscale.com/kb/1337/policy-syntax#hosts) that maps to an IP address.

### [`accept`](https://tailscale.com/kb/1337/policy-syntax\#accept)

Specifies zero, one, or more usernames to disallow on the `dst` host without requiring an additional check. Refer to [action `accept`](https://tailscale.com/kb/1337/policy-syntax#action-1).

### [`check`](https://tailscale.com/kb/1337/policy-syntax\#check)

Specifies zero, one, or more usernames to disallow on the `dst` host if the `src` user passes an additional check. Refer to [action `check`](https://tailscale.com/kb/1337/policy-syntax#action-1).

### [`deny`](https://tailscale.com/kb/1337/policy-syntax\#deny)

Specifies zero, one, or more usernames to disallow on the `dst` host (under any circumstances).

## [IP sets](https://tailscale.com/kb/1337/policy-syntax\#ip-sets)

An IP set is a way to manage groups of IP addresses. It can encapsulate a collection of IP addresses, CIDRs, hosts, autogroups, and other IP sets. The primary benefit of IP sets is that they let you group multiple network parts into a single collection, enabling you to apply access control policies to the collection rather than the individual IP addresses, hosts, or subnets.

Refer to the [IP sets documentation](https://tailscale.com/kb/1387/ipsets).

## [Network policy options](https://tailscale.com/kb/1337/policy-syntax\#network-policy-options)

Network policy optionsare available for [all plans](https://tailscale.com/pricing).

In addition to access rules, the tailnet policy file includes a few network-wide policy settings for specialized purposes. Most networks should never need to specify these.

### [`derpMap`](https://tailscale.com/kb/1337/policy-syntax\#derpmap)

The `derpMap` section lets you add [custom DERP servers](https://tailscale.com/kb/1118/custom-derp-servers) to your network, which your devices will use as needed to relay traffic. You can also use this section to disable using Tailscale-provided DERP servers. For example, you might want to disable tailnet-provided DERP servers to meet corporate compliance requirements. Refer to [running custom DERP servers](https://tailscale.com/kb/1118/custom-derp-servers) for more information.

### [`disableIPv4`](https://tailscale.com/kb/1337/policy-syntax\#disableipv4)

Instead of the `disableIPv4` field, it is recommended to use the `disable-ipv4` node attribute as described in [CGNAT conflicts](https://tailscale.com/kb/1023/troubleshooting#cgnat-conflicts).

The `disableIPv4` field (if set to `true`) stops assigning Tailscale IPv4 addresses to your devices. When IPv4 is disabled, all devices in your network receive exclusively IPv6 Tailscale addresses. Devices that do not support IPv6 (for example, systems that have IPv6 disabled in the operating system) will be unreachable. This option is intended for users with a pre-existing conflicting use of the `100.64.0.0/10` carrier-grade NAT address range.

### [`OneCGNATRoute`](https://tailscale.com/kb/1337/policy-syntax\#onecgnatroute)

The `OneCGNATRoute` field controls the routes that Tailscale clients generate.

Tailscale clients can have either:

- One large `100.64/10` route to avoid churn in the routing table as devices go online and offline. (The churn is [disruptive](https://bugs.chromium.org/p/chromium/issues/detail?id=1076619) to Chromium-based browsers on macOS.)
- Fine-grained `/32` routes.

The possible values for `OneCGNATRoute` are:

- An empty string or not provided: Use default heuristics for each platform.
  - For all platforms (other than macOS), Tailscale adds fine-grained `/32` routes for each device.
  - On macOS (for Tailscale v1.28 or later), Tailscale adds one `100.64/10` route. Tailscale won't use one `100.64/10` route if other interfaces also route IP addresses in that range.
- `"mac-always"`: macOS clients always add one `100.64/10` route.
- `"mac-never"`: macOS clients always add fine-grained `/32` routes.

### [`randomizeClientPort`](https://tailscale.com/kb/1337/policy-syntax\#randomizeclientport)

The `randomizeClientPort` field (if set to `true`) makes devices prefer a random port for WireGuard traffic over the default static port `41641`. You should only use the `randomizeClientPort` field as a workaround for some buggy firewall devices after consulting with Tailscale ( [support](https://tailscale.com/contact/support)).

Last updated Jun 30, 2025