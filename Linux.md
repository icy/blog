## Linux linux...

### Sharing uid/gid between multiple Linux users

It's very possible to create multiple  users and groups those share the same uid/gid.
Seriously this may confuse everyone but sometimes you will really need to do that.
I won't tell my "why"; but I share how the thing would be done.

1. For groups, you need to modify 3 files `/etc/group`, `/etc/group-` and  `/etc/gshadow`.
   What to modify?  Well, you create a new line which contains exactly some same contents
   as the line that includes the source group, and update the line with your new group name.
   For example,
   let's say we have an original line `group:bill.gates:x:1000:bill.gates` that
   indicates the group `bill.gates` includes an user named `bill.gates`. Now
   create new line `group:bill.fences:x:1000:bill.gates`, and after saving the file
   `/etc/group`, you will have two groups `bill.fences`  and `bill.gates`, both
   share the same gid=1000, and both includes the same user `bill.gates` :)
1. For users, you follow the same id, but changes land on 3 files `/etc/passwd`,
   `/etc/passwd-` and `/etc/shadow`.

Please mind the order of new entries. The very first line wins and some application which
works with `uid` may only print the first username what matches the `uid`.

### Fixing my custom dns resolver after system upgrades

Before continuing these commands, I have to fix `/etc/resolv.conf` to use some temporary DNS resolver.
The change will be reverted once `mdns` can be up and running.

```
$ sudo su -
$ find /usr/lib/ruby -type d -exec chmod o+x,g+x -R {} \;
$ find /usr/lib/ruby -type f -exec chmod o+r,g+r -R {} \;
$ su - dns -s /bin/bash
$ gem install --user-install eventmachine --version "~> 1.0.0"
$ gem install --user-install rubydns --version "0.6.7"
$ exit
$ systemctl restart mdns
```
