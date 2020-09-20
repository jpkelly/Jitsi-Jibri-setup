# Jitsi with Jibri setup on AWS EC2 Ubuntu Server 18.04 
Setting up Jitsi-Meet with Jibri on Ubuntu Server 18.04

## Set up Jitsi-Meet as described here.
https://github.com/jpkelly/Jitsi-setup/

## On Jitsi-Meet server configure Prosody
### Edit `/etc/prosody/conf.avail/<FQDN>.cfg.lua`
Add MUC component
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
Add recorder virtual host
```
VirtualHost "recorder.<FQDN>"
  modules_enabled = {
    "ping";
  }
  authentication = "internal_plain"
```

### Add Jibri users
```
prosodyctl register jibri auth.<FQDN> jibriauthpass
prosodyctl register recorder recorder.<FQDN> jibrirecorderpass
```

### Jitsi-Meet config
Edit `/etc/jitsi/meet/yourdomain-config.js`
```
fileRecordingsEnabled: true, // If you want to enable file recording
liveStreamingEnabled: true, // If you want to enable live streaming
hiddenDomain: 'recorder.<FQDN>',
```

## Jicofo
Edit `/etc/jitsi/jicofo/sip-communicator.properties` (must have the following)
```
org.jitsi.jicofo.BRIDGE_MUC=JvbBrewery@internal.auth.<FQDN>
org.jitsi.jicofo.auth.URL=XMPP:<FQDN>
org.jitsi.jicofo.jibri.BREWERY=JibriBrewery@internal.auth.<FQDN>
org.jitsi.jicofo.jibri.PENDING_TIMEOUT=90
```

## Dropbox Integration
https://jitsi.github.io/handbook/docs/dev-guide/dev-guide-web-integrations

## AWS S3 Integration
