## Linux linux...

### Sharing uid/gid between multiple Linux users

_Updated 2026-08-12:_ Podman-6.0 will complain when there are duplicated uid/gid on the system.

> ERRO[0000] running /usr/bin/newuidmap 134121 0 1001 1 1 10000 65536 65537 10000 65536:
> newuidmap: write to uid_map failed: Invalid argument
> Error: fatal error, invalid internal status, unable to create a new pause process:
> cannot set up namespace using "/usr/bin/newuidmap": exit status 1.
> Try running "podman system migrate" and if that doesn't work reboot to recover

To get rid of this error, you can edit the two files `/etc/subgid` and `/etc/subuid`
and remove the `duplicated` entries: If two users A and B whose  `uid/gid` are the same,
only A (or B)'s entry should present in `/etc/subuid`  and `/etc/subgid`.

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
1. For users, you follow the same way, but changes land on 3 files `/etc/passwd`,
   `/etc/passwd-` and `/etc/shadow`.

Please mind the order of new entries. The very first line wins and some application which
works with `uid` may only print the first username what matches the `uid`.

### Fixing my custom dns resolver after system upgrades

Before continuing these commands, I have to fix `/etc/resolv.conf` to use some temporary DNS resolver.
The change will be reverted once `mdns` can be up and running. For some reason (:D) system umask was set to 077
and new installation of Ruby plugins at system level was inaccessible by normal users.

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
