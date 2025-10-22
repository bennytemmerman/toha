---
title: Splunk searches
weight: 201
menu:
  notes:
    name: GUI
    identifier: notes-splunk-gui
    parent: notes-splunk
    weight: 21
---
<div style="display: block; width: 100%; max-width: none;">
{{< note title="Debug refresh:" >}}
If you change layout, configuration, or app files in Splunk, you can reload the resources through the web GUI, which is faster than restarting Splunk. 
If you still don’t see your changes, you might need to restart Splunk completely to clear cached data and make sure all updates are applied.
```bash
https://<host>:<port>/debug/refresh
```
- __JavaScript (JS) files:__ Updates to scripts that run in the browser UI.
- __XML and HTML files:__ Changes to the layout, structure, or look of the interface.
- __Configuration files:__ Settings that control how Splunk works.
- __App files:__ Other resources and assets within your Splunk apps that have been changed.
  
[resource](https://dev.splunk.com/enterprise/docs/developapps/manageknowledge/assetcaching/?_gl=1*16bjotw*_gcl_aw*R0NMLjE3NTk5ODg2ODQuQ2owS0NRandsNWpIQmhESEFSSXNBQjBZcWp3NHJhQk1uUUVyeEc5OXcweGt0TU0tTWl6cjB2N05lV1RiYi1kSGt4endkOHZWQ0NFVy1YUWFBa2l3RUFMd193Y0I.*_gcl_au*NDc3MjU1Nzc0LjE3NTg2MDcyNjY.*FPAU*NDc3MjU1Nzc0LjE3NTg2MDcyNjY.*_ga*Mzg3OTA2MjM0LjE3NDI0NjQ3MzU.*_ga_5EPM2P39FV*czE3NjExMTYwNjgkbzkzJGcwJHQxNzYxMTE2MDY4JGo2MCRsMCRoMTIyNDM2OTU4MQ..*_fplc*RXhXckVEOVBuVjZxRzQ0M3RhJTJGWGVmVXJVdEFkRHFOUlZFdHAlMkZXbzRreHZBSEpDMVclMkIzN2JtMkJwMUZMTGpoamtEOGo5NjFGQnhhalZBSGpzNDdZSHZqY2syRjc0WFJXWTFISUNyeTcxOEtTWnZEZEQxeER0QyUyRjhzUndLUGclM0QlM0Q.)
{{< /note >}}
{{< note title="SSO bypass local account login" >}}
I’ve seen several setups where SSO is turned on but not set up correctly, which can lead to users having the wrong permissions. Even if local accounts are still enabled, the login URL will redirect to authenticate with SAML or another service. To log in with a local account instead, just add this to the end of the original URL: /en-US/account/login?loginType=splunk
```bash
https://<host>:<port>/en-US/account/login?loginType=splunk
```  
[resource](https://splunk.my.site.com/customer/s/article/How-to-login-into-Splunk-using-local-Splunk-accounts-after-configuring-SAML)
{{< /note >}}
</div>