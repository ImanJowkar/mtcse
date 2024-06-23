# MTCSE (MiKroTik Certified Security Engineer)

## Steps to secure Mikrotik:

* `always update your router-os to the latest stable release.`


* `If the bandwidth test server is not being used, it should be disabled.`
```
tool/bandwidth-server/set enabled=no
```

* `disable mac ping `
```
/tool mac-server ping
set enabled=no
```

* `if you're not using mac-telnet then disable it.`


* `Always generate a new user with administrative privileges and remove the default admin user.`


* `if you're not using RoMon then disable it. by default this service is disable.`



