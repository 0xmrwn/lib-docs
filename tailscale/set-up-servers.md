# Setting up a server on your Tailscale network

If you're setting up servers on Tailscale, we recommend you use an [auth key](https://tailscale.com/kb/1085/auth-keys) to provision the server, and a [tag](https://tailscale.com/kb/1068/tags) to restrict its access. You can also set up [Tailscale SSH](https://tailscale.com/kb/1193/tailscale-ssh) to access your servers.

Here's how to set up a server in Tailscale:

1. Create a new [tag](https://tailscale.com/kb/1068/tags) in your tailnet for the type of shared resource you are managing. For example, you can use the tag `server` for your servers, `prod` or `test` for your environments, and `front-end` for grouping of other resources that you maintain.

To create a tag, modify the [tailnet policy file](https://tailscale.com/kb/1018/acls) to specify the owner of the tag, which is the team or user who can use that tag. You can use an existing tag for all servers. If you're also setting up Tailscale SSH, we recommend using a new tag.







```json
{
     "tagOwners": {
       "tag:server": ["alice@example.com"]
     }
}

```

2. Write [access rules](https://tailscale.com/kb/1337/policy-syntax#acls) in the tailnet policy file which:


   - Allow the desired sources to reach the tagged resources
   - If you're also setting up Tailscale SSH, allow the desired sources to reach the tagged resources using Tailscale SSH

```json
{
  "grants": [\
    {\
      "src": ["group:sre"],\
      "dst": ["tag:server"],\
      "ip": ["*"]\
    }\
  ],
  "ssh": [\
    {\
      "action": "accept",\
      "src": ["group:sre"],\
      "dst": ["tag:server"],\
      "users": ["ubuntu", "root"]\
    }\
  ]
}

```

3. Generate an [authentication key](https://tailscale.com/kb/1085/auth-keys) to automatically connect servers to your network. Select the tag or tags you wish to use for your servers as part of this auth key.


   - If you're authenticating more than one server, use a reusable auth key. Or, for long-lived auth keys, set up an [OAuth client with the scope `auth_keys`](https://tailscale.com/kb/1215/oauth-clients#generating-long-lived-auth-keys).
   - If you're authenticating ephemeral workloads like containers or functions, use an ephemeral key.
   - If your tailnet has [device approval](https://tailscale.com/kb/1099/device-approval) enabled, and you only intend to use that to approve end-user devices, use a pre-authorized auth key.

![Tailscale's auth key generation page](https://tailscale.com/_next/image?url=%2F_next%2Fstatic%2Fmedia%2Fgenerate-auth-key.a0369d5c.png&w=750&q=75)

The **Pre-approved** option will only display in the dialog if [device approval](https://tailscale.com/kb/1099/device-approval) is enabled in your Tailscale network.

Currently, if your client node is provisioned with an [authentication key](https://tailscale.com/kb/1085/auth-keys), you cannot use [check mode](https://tailscale.com/kb/1193/tailscale-ssh#check-mode) when
establishing a Tailscale SSH connection using the node as a source.

4. When you provision a new server, [install](https://tailscale.com/download) and connect to Tailscale manually or as part of your automation tooling. Make sure to specify the auth key including the tags you want, and to enable Tailscale SSH.







```shell
tailscale up --auth-key=$TS_AUTHKEY

```



If you want to specify a particular machine name for your server to use with [MagicDNS](https://tailscale.com/kb/1081/magicdns), then also specify `--hostname`:







```shell
tailscale up --auth-key=$TS_AUTHKEY --hostname=$TS_HOSTNAME

```



If the auth key was not generated with tags, then also specify `--advertise-tags`:







```shell
tailscale up --auth-key=$TS_AUTHKEY --advertise-tags=<tags>

```



If you want to enable Tailscale SSH, then also specify `--ssh`:







```shell
tailscale up --auth-key=$TS_AUTHKEY --ssh

```

5. To access your servers over Tailscale, you can connect to them like any other device in your tailnet.

If you have Tailscale SSH set up, you can connect to your severs using [Tailscale SSH](https://tailscale.com/kb/1193/tailscale-ssh) or [Tailscale SSH Console](https://tailscale.com/kb/1216/tailscale-ssh-console).


Last updated Jul 1, 2025