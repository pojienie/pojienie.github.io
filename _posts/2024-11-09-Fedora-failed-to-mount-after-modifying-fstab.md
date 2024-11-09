# introduction

Around an year ago, I bought two USB SSDs in order to backup my stuff. I didn't want to mount those SSDs by myself every time so I modified `/etc/fstab` and let Linux mount it for me. However, ever since that day, I was no longer able to login to my user without `mount -a` first. I knew why I couldn't login - it was because my `/home` was on another partition - also supposed to be mounted by Linux automatically because there was an entry in `fstab` file. However, what I didn't understand was: why Linux wasn't able to mount but I could?

The workaround worked for me so I never really looked into it.

# solution that worked for me

I used the workaround every time I rebooted my machine. Until around 2 weeks ago, I suddenly thought that it might be because of SELinux conext. I was using Fedora which was made by Red Hat and I knew all Red Hat family Linux distro had SELinux enabled. Thus, I used `ls -Z /etc` and its output was:

```
...
system_u:object_r:etc_t:s0 fprintd.conf
unconfined_u:object_r:user_home_t:s0 fstab
system_u:object_r:etc_t:s0 fuse.conf
...
```

Ah ha, the SELinux context was wrong! I used `chcon` to change the SELinux context, specifically:

```
chcon -t etc_t /etc/fstab 
chcon -u system_u /etc/fstab 
```

Today, I finally needed to reboot and I was really happy to see that I was able to login without switching to a terminal session first!
