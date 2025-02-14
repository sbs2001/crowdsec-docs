# Plugins

## Architecture

CrowdSec supports various plugins. Under the hood, the main CrowdSec process dispatches the plugins as GRPC services. This means that plugins can be written in any language.

Plugins are owned by a `profile`. 

Currently only `notification` plugins are supported. Whenever CrowdSec receives any alert, if this alert satisfies the owner profile then the same alert will be dispatched to such plugin.

[See](https://github.com/crowdsecurity/crowdsec/blob/plugins/pkg/protobufs/notifier.proto) the GRPC protocol for `notification` plugins.

## Plugin Discovery

Plugins are discovered from the directories specified in `/etc/crowdsec/config.yaml`. 

```yaml
#/etc/crowdsec/config.yaml
.....
config_paths:
  plugin_dir: /etc/crowdsec/plugins/
  notification_dir: /etc/crowdsec/notifications/
.....
```


### `plugin_dir`: 

Path to directory where the plugin binaries/scripts are located. 

### `notification_dir`:

Path to directory where configuration files for `notification` plugins are kept.


**Important:** CrowdSec rejects the plugins if one of the following is true :
1. plugin is not owned by the root user and root group.
2. plugin is world-writable. 

**Important:** Plugin binaries/scripts must be named like `<plugin_type>-<plugin_subtype>` eg "notification-slack"

## Registering plugin to profile: 

After discovering the plugins, CrowdSec registers the plugins to the profiles. Here's a profile which sends alerts to plugin named `slackreport`. 

```yaml
name: default_ip_remediation
#debug: true
filters:
 - Alert.Remediation == true && Alert.GetScope() == "Ip"
decisions:
 - type: ban
   duration: 4h
on_success: break
notifications:
 - slackreport

```

**Note:** In this case CrowdSec will map the plugin `slackreport` to the plugin config which has `name=slackreport`. See next section for more details.

## Notification plugin configuration:

Following are the fields CrowdSec main process can interpret. 
```yaml
type:
name:
format:
group_wait:
group_threshold:
max_retry:
timeout:

```
{% raw %}
### `type` : 

Required. Type of plugin, eg "slack"

### `name` : 

Required. Name of this config  eg "slackreport". This should match with registered name in profile.

### `format` :

Required. go template, which is fed a slice of `Alert` objects. eg "Received {{.len}} alerts"

### `group_wait` :

Optional. duration to wait collecting alerts before sending to this plugin, eg "30s"

### `group_threshold` :

Optional. if alerts exceed this, then the plugin will be sent the message. eg "10"

### `max_retry` :

Optional. the number of tries to attempt to send message to plugins in case of error.

### `timeout` :

Optional. duration to wait for a response from the plugin before considering this attempt a failure. eg "10s"

{% endraw %}

See  definition of alert [here](https://pkg.go.dev/github.com/crowdsecurity/crowdsec@v1.1.1/pkg/models#Alert).

You can define other plugin specific fields too. eg `webhook` field for a slack plugin. 

## Dispatching configuration: 

CrowdSec main process feeds the configuration files to the plugins via GRPC. It determines where to send the config via the value of `type`  field in config file.

Example: plugin binaries with name="notification-slack" will be fed all config files where `type==slack`# Understanding Plugins

## Architecture

CrowdSec supports various plugins. Under the hood the main CrowdSec process dispatches the plugins as GRPC services. This means that plugins can be written in any language.

Currently only `notification` plugins are supported. Whenever CrowdSec receives any alert, the same alert will be dispatched to such plugins.

[See](https://github.com/crowdsecurity/crowdsec/blob/plugins/pkg/protobufs/notifier.proto) the GRPC protocol for `notification` plugins.

## Plugin Discovery

Plugins are discovered from the directories specified in `/etc/crowdsec/config.yaml`. 

```yaml
#/etc/crowdsec/config.yaml
.....
config_paths:
  plugin_dir: /etc/crowdsec/plugins/
  notification_dir: /etc/crowdsec/notifications/
.....
```


### `plugin_dir`: 

Path to directory where the plugin binaries/scripts are located. 

### `notification_dir`:

Path to directory where configuration files for `notification` plugins are kept.


**Important:** CrowdSec rejects the plugins if one of the following is true :
1. plugin is not owned by root user and root group.
2. plugin is world-writable. 

**Important:** Plugin binaries/scripts must be named like `<plugin_type>-<plugin_subtype>` eg "notification-slack"

## Notification plugin configuration:

Following are the fields CrowdSec main process can interpret. 
```yaml
type:
name:
format:
group_wait:
group_threshold:
max_retry:
timeout:

```
{% raw %}
### `type` : 

Required. Type of plugin, eg "slack"

### `name` : 

Required. Name of this config  eg "slackreport"

### `format` :

Required. go template, which is fed a slice of `Alert` objects. eg "Received {{.len}} alerts"

### `group_wait` :

Optional. duration to wait collecting alerts before sending to this plugin, eg "30s"

### `group_threshold` :

Optional. if alerts exceed this, then the plugin will be sent the message. eg "10"

### `max_retry` :

Optional. number of tries to attempt to send message to plugins in case of error.

### `timeout` :

Optional. duration to wait for response from plugin before considering this attempt a failure. eg "10s"

{% endraw %}

See  definition of alert [here](https://pkg.go.dev/github.com/crowdsecurity/crowdsec@v1.1.1/pkg/models#Alert).

You can define other plugin specific fields too. eg `webhook` field for a slack plugin. 

## Dispatching configuration: 

CrowdSec main process feeds the configuration files to the plugins via GRPC. It determines where to send the config via the value of `type`  field in config file.

Example: plugin binaries with name="notification-slack" will be fed all config files where `type==slack`