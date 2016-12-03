# About

ServerMonitor is a C++14 command line program for determining if a server or a service on a server is available (up) or not (down). It can monitor using four methods:

1. HTTP(s) - the server must output status 200 or redirect to a page that outputs status 200
2. Port - the server must be listening on this port and accept a connection
3. Ping - the server must respond to a ping
4. Custom Command - a command can be run to provide custom logic to determine if a server is running. Exit code 0 is up, and anything else is down.

ServerMonitor checks each server in parallel, so the total time to check all servers will only be as long as the slowest server, instead of the total of all monitor times.

# Configuration

The program is ran like so:

    ServerMonitor <input_config.json> <output_status.json>

There are only two command line arguments:

1. Input config - this is a JSON file with all configuration options specified. See below for examples.

2. Output status - after checking all servers, a JSON status report will get generated, which can then be consumed by any program that can read JSON. See [status.html](status.html) as an example static HTML page that renders the JSON file via jQuery.

# Examples

To monitor Apple's website:

```json
{
  "servers": [
    {
      "name": "Apple Website",
      "url": "http://apple.com"
    }
  ]
}
```

To monitor GitHub's SSH port:

```json
{
  "servers": [
    {
      "name": "GitHub SSH",
      "host": "github.com",
      "port": 22
    }
  ]
}
```

To monitor Google.com via ping:

```json
{
  "servers": [
    {
      "name": "Google",
      "ping": "google.com"
    }
  ]
}
```

To monitor via a custom command, for example SSH into a server and check for a file's existance:

```json
{
  "servers": [
    {
      "name": "My Server",
      "cmd": "ssh my.server test -f myfile.txt"
    }
  ]
}
```

# Actions

Currently the above examples don't provide any type of notification of when a server goes up or down. For this you must use the `actions` key. Currently only a command can be run.

For example, to display a notification in macOS:

```json
{
  "actions": {
    "notif": {
      "cmd": "osascript -e 'display notification \"Server is {{STATUS}}\" with title \"{{name}}\"'"
    }
  },
  "servers": [
    {
      "name": "Apple Website",
      "url": "http://apple.com",
      "action": "notif"
    }
  ]
}
```

As used above, actions can use the following case-sensitive variables:

| Variable   | Description    |
| ---------- | -------------- |
| {{name}}   | Server name    |
| {{status}} | "up" or "down" |
| {{Status}} | "Up" or "Down" |
| {{STATUS}} | "UP" or "DOWN" |

# Advanced Options

Advanced options can be set at the root level (along side `actions` and `servers`) and/or overridden for each individual server.

| Option | Type | Description | Default |
| --- | --- | --- | --- |
| timeout | Integer | The timeout in seconds to wait for a response. | 5 |
| verifypeer | Boolean | Enable or disable CURL's [VERIFYPEER](https://curl.haxx.se/libcurl/c/CURLOPT_SSL_VERIFYPEER.html) option. Useful for websites with self-signed or expired SSL certificates. | true |

Example:

```json
{
  "servers": [
    {
      "name": "Apple Website",
      "url": "https://apple.com",
      "timeout": 30,
      "verifpeer": false
    }
  ]
}
```

# Scheduling

Here's an example launchd plist for macOS for running ServerMonitor every minute:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>Label</key>
	<string>com.kainjow.servermonitor</string>
	<key>ProgramArguments</key>
	<array>
		<string>/Users/you/ServerMonitor/ServerMonitor</string>
		<string>/Users/you/ServerMonitor/config.json</string>
		<string>/Users/you/ServerMonitor/status.json</string>
	</array>
	<key>StartInterval</key>
	<integer>60</integer>
</dict>
</plist>
```

Or with cron:

    * * * * * ~/ServerMonitor/ServerMonitor ~/ServerMonitor/config.json ~/ServerMonitor/status.json

# Building

Dependencies:

- [CMake](https://cmake.org) 3.1 or later
- CURL

First make sure you cloned recursively:

    git clone --recursive https://github.com/kainjow/ServerMonitor.git

Then run `make`.
