## Description

As an engineer I need to tune my laptop to run at its best performance. Why? The answer is `why not`. 

## Specification

My laptop is pretty old. It's at least 7 years old. 
(It's X240, I bought it from the 2nd market.)
Its memory capacity is 8GB. And cpu? 
It's  `Intel(R) Core(TM) i5-4210U CPU @ 1.70GHz` (2 cores x 2 threads)?
This is just enough for daily web surfing... until it isn't.

## CPU power policy

As the laptop CPU can be adjusted to use much or less power, here comes CPU Power policy. 
When my browser is running the laptop really needs CPU Power policy set to `Maximum Speed`.
This would be fine as long as the external power source is available. The CPU fan
will work accordingly :)

## Slow down my applications

```
$ cpulimit -l 20 terragrunt --terragrunt-parallelism 1 apply
$ nice -n 19 terraform -parallelism 1
```

## Enlightenment vs xfce4 vs YourWarName

### Since 2011

I'm using Enlightenment. I've been using it. Since 2011 or even before that, I don't remember =))

My Enlightenment setup is a nightmare, in some situation it consumes quite a lot of CPU. Switching quickly to xfc4
really helps to save the Earth.

Why don't I uninstall Enlightenment? I think it'd be some problem hidden with my setup, and I haven't found it.
What I tried in the past

* Pinned version: Use Englightenment 0.23.x instead of the latest version. It worked for a while.
* Fixed all troublesome keyboard/input issues (yeah my keyboard/touchpad is kinda broken and it generates
quite a lot of annoying events and they can slow down the system at the same time.)
* Traced some problems down to... nowhere.

### Cpulimit

Sometimes Enlightenment uses so much cpu. It's 50% from htop output. Looks like an unknown issue with vesa
and though admitted no root cause/fix has been found. An easy work-around is to run the daemon with some cpu limit

```
exec dbus-launch  cpulimit -l 40 enlightenment_start
```

Using 30 or lower number will not work. It will force Enlightenment to behave crazily: Starting with 20, 
it's impossible to login. With 30, (I) can log in, slowly, and when the desktop environment is locked (ie when I'm
away from my laptop for 5 minutes), the login screen will lead to another login screen , and it ends up with
some wayland or alike windows inside enlightenment :) That's the best bug I've found. I think I took a picture
of this but yeah, after switching to `-l 40` everything works fine and I haven't forgotten the issue with 30.

## Firefox

### cpulimit

```
exec dbus-launch cpulimit -l 200 firefox --class FirefoxNoXXX -ProfileManager -no-remote
```

### Config

Most of the time I can't read/view two tabs at the same time, 
and my eyes wouldn't be faster than any cpu unit. 
When I slow down firefox I still see the same results. 
It's interesting that the defaults settings are very high 
and really make firefox hang on my laptop.

* dom.ipc.plugins.processLaunchTimeoutSecs=1
* dom.ipc.processCount=1
* dom.ipc.processCount.webIsolated=1
* dom.ipc.processCount.webLargeAllocation=1
* dom.ipc.processPrelaunch.fission.number=1
* javascript.options.concurrent_multiprocess_gcs.cpu_divisor=1

FIXME: It's said on some threads that Google's browser doesn't 
have any option to reduce the number of cores they are on. 
It may be wrong but only would I use such browser for some rare situation,
so not a big deal...

### Addons

The world has changed. Web owners don't want to waste their money and they force
web browsers (aka their clients) to use more and more resources. Some web owners
don't serve if you don't have javascript, the cpu&memory killer.

So let make that behavior default.

* NoScript
* Cookie Autodelete
* AdBlocker Ultimate
* uBlock Origin
* Ghostery (automatically rejects almost all cookie consent questions in EU)

Note: I have AdGuard AdBlocker on firefox mobile version

### A screenshot

Thanks to `pstree`

```
├─firefox─┬─Isolated Servic───20*[{Isolated Servic}]
│         ├─Isolated Web Co───29*[{Isolated Web Co}]
│         ├─Privileged Cont───19*[{Privileged Cont}]
│         ├─RDD Process───7*[{RDD Process}]
│         ├─Socket Process───4*[{Socket Process}]
│         ├─Utility Process───6*[{Utility Process}]
│         ├─3*[Web Content───16*[{Web Content}]]
│         ├─WebExtensions───21*[{WebExtensions}]
│         └─94*[{firefox}]
```

## When it is slow...

* htop. Look at cpu usage, which is often indicating the source
  of the problem on my laptop. It's either Enlightenment or Firefox
* iostat or vmstat for fun :)

## Uprecords

```
     1    93 days, 22:14:52 | Linux 5.10.5-arch1-1      Mon Mar  1 07:31:59 2021
     2    66 days, 09:53:32 | Linux 6.2.8-arch1-1       Fri Apr 14 20:39:38 2023
->   3    46 days, 04:32:58 | Linux 6.10.6-arch1-1      Fri Aug 23 16:27:14 2024
     4    43 days, 14:21:48 | Linux 6.5.7-arch1-1       Fri Dec  1 20:10:17 2023
     5    42 days, 02:31:50 | Linux 5.10.5-arch1-1      Sun Jan 17 20:03:24 2021
     6    33 days, 01:09:50 | Linux 6.1.4-arch1-1       Wed Feb  8 15:55:10 2023
```

## The tasks this laptop has been serving...

- [x] A coin miner during covid time. For fun, of course, net profit is negative :)
- [ ] Provide the main media system for my little family :D Mostly music
- [ ] A main source of income for many years :D
- [x] An image server. Now I have to buy a NAS device
- [ ] A backup server for android devices 
