# Jitsi with Jibri setup on AWS EC2 Ubuntu Server 18.04 
Setting up Jitsi-Meet with Jibri on Ubuntu Server 18.04

## Set up Jitsi-Meet as described here.
https://github.com/jpkelly/Jitsi-setup/

## On Jitsi-Meet server 
### Configure Prosody 
Edit `/etc/prosody/conf.avail/<FQDN>.cfg.lua`

Add MUC component (already exists?)
```
-- internal muc component, meant to enable pools of jibri and jigasi clients
Component "internal.auth.dev.vevomo.com" "muc"
    storage = "memory"
    modules_enabled = {
      "ping";
    }
    admins = { "focus@auth.<FQDN>", "jvb@auth.<FQDN>" }
    muc_room_locking = false
    muc_room_default_public_jids = true
```
Add recorder virtual host (after guest VirtualHost)
```
VirtualHost "recorder.<FQDN>"
  modules_enabled = {
    "ping";
  }
  authentication = "internal_plain"
```

### Add Jibri users
```
prosodyctl register jibri auth.<FQDN> <jibriauthpass>
prosodyctl register recorder recorder.<FQDN> <jibrirecorderpass>
```

### Jitsi-Meet config
Edit `/etc/jitsi/meet/<FQDN>-config.js`
```
fileRecordingsEnabled: true, // If you want to enable file recording
liveStreamingEnabled: true, // If you want to enable live streaming
hiddenDomain: 'recorder.<FQDN>',
```

### Jicofo
Edit `/etc/jitsi/jicofo/sip-communicator.properties` (must have the following)
```
org.jitsi.jicofo.BRIDGE_MUC=JvbBrewery@internal.auth.<FQDN>
org.jitsi.jicofo.auth.URL=XMPP:<FQDN>
org.jitsi.jicofo.jibri.BREWERY=JibriBrewery@internal.auth.<FQDN>
org.jitsi.jicofo.jibri.PENDING_TIMEOUT=90
```

### Dropbox Integration
https://jitsi.github.io/handbook/docs/dev-guide/dev-guide-web-integrations

### AWS S3 Integration

---

## Jibri Server
snd-aloop module needs to be enabled
On AWS you need to update your kernel. You can refer to these steps and then continue with the Jibri Installation guide to finalize your installation:

### Update package list
apt update
// Install sound module , which is in package linux-image-extra-virtual
`apt -y install linux-image-extra-virtual`
// In between it prompts to update maintainer version, select first option)

Edit default grub file (to make aws kernel generic):
`vim /etc/default/grub` and set
`GRUB_DEFAULT=“1>2”`
Run `update-grub`
And then reboot now to take this in effect
// reboot machine (as package version was different than linux kernel version)

above did not work
use
https://meetrix.io/blog/aws/changing-default-ubuntu-kernel.html

purge aws kernel so update does not break grub boot order
`apt-get purge 'linux-image-*aws*'`

set grub line back to default in `/etc/default/grub`
`GRUB_DEFAULT=0`
`update-grub`
`reboot`




modprobe snd-aloop
// Check aloop correctly exists
`lsmod | grep aloop`

// make it automatically start
`echo “snd-aloop” >> /etc/modules`


## These files need to be in /usr/local/bin?

https://github.com/jpkelly/emrah-buster-templates/tree/master/machines/eb-jibri-template/usr/local/bin
