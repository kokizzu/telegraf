# Windows Performance Counters Input Plugin

This plugin produces metrics from the collected
[Windows Performance Counters][perf_counters].

⭐ Telegraf v0.10.2
🏷️ system
💻 windows

[perf_counters]: https://learn.microsoft.com/en-us/windows/win32/perfctrs/about-performance-counters

## Global configuration options <!-- @/docs/includes/plugin_config.md -->

In addition to the plugin-specific configuration settings, plugins support
additional global and plugin configuration settings. These settings are used to
modify metrics, tags, and field or create aliases and configure ordering, etc.
See the [CONFIGURATION.md][CONFIGURATION.md] for more details.

[CONFIGURATION.md]: ../../../docs/CONFIGURATION.md#plugins

## Configuration

```toml @sample.conf
# Input plugin to counterPath Performance Counters on Windows operating systems
# This plugin ONLY supports Windows
[[inputs.win_perf_counters]]
  ## By default this plugin returns basic CPU and Disk statistics. See the
  ## README file for more examples. Uncomment examples below or write your own
  ## as you see fit. If the system being polled for data does not have the
  ## Object at startup of the Telegraf agent, it will not be gathered.

  ## Print All matching performance counters
  # PrintValid = false

  ## Whether request a timestamp along with the PerfCounter data or use current
  ## time
  # UsePerfCounterTime = true

  ## If UseWildcardsExpansion params is set to true, wildcards (partial
  ## wildcards in instance names and wildcards in counters names) in configured
  ## counter paths will be expanded and in case of localized Windows, counter
  ## paths will be also localized. It also returns instance indexes in instance
  ## names. If false, wildcards (not partial) in instance names will still be
  ## expanded, but instance indexes will not be returned in instance names.
  # UseWildcardsExpansion = false

  ## When running on a localized version of Windows and with
  ## UseWildcardsExpansion = true, Windows will localize object and counter
  ## names. When LocalizeWildcardsExpansion = false, use the names in
  ## object.Counters instead of the localized names. Only Instances can have
  ## wildcards in this case. ObjectName and Counters must not have wildcards
  ## when this setting is false.
  # LocalizeWildcardsExpansion = true

  ## Period after which counters will be reread from configuration and
  ## wildcards in counter paths expanded
  # CountersRefreshInterval="1m"

  ## Accepts a list of PDH error codes which are defined in pdh.go, if this
  ## error is encountered it will be ignored. For example, you can provide
  ## "PDH_NO_DATA" to ignore performance counters with no instances. By default
  ## no errors are ignored You can find the list here:
  ##   https://github.com/influxdata/telegraf/blob/master/plugins/inputs/win_perf_counters/pdh.go
  ## e.g. IgnoredErrors = ["PDH_NO_DATA"]
  # IgnoredErrors = []

  ## Maximum size of the buffer for values returned by the API
  ## Increase this value if you experience "buffer limit reached" errors.
  # MaxBufferSize = "4MiB"

  ## NOTE: Due to the way TOML is parsed, tables must be at the END of the
  ## plugin definition, otherwise additional config options are read as part of
  ## the table

  # [[inputs.win_perf_counters.object]]
    # Measurement = ""
    # ObjectName = ""
    # Instances = [""]
    # Counters = []
    ## Additional Object Settings
    ##   * IncludeTotal: set to true to include _Total instance when querying
    ##                   for all metrics via '*'
    ##   * WarnOnMissing: print out when the performance counter is missing
    ##                    from object, counter or instance
    ##   * UseRawValues: gather raw values instead of formatted. Raw values are
    ##                   stored in the field name with the "_Raw" suffix, e.g.
    ##                   "Disk_Read_Bytes_sec_Raw".
    # IncludeTotal = false
    # WarnOnMissing = false
    # UseRawValues = false

  ## Processor usage, alternative to native, reports on a per core.
  # [[inputs.win_perf_counters.object]]
    # Measurement = "win_cpu"
    # ObjectName = "Processor"
    # Instances = ["*"]
    # UseRawValues = true
    # Counters = [
    #   "% Idle Time",
    #   "% Interrupt Time",
    #   "% Privileged Time",
    #   "% User Time",
    #   "% Processor Time",
    #   "% DPC Time",
    # ]

  ## Disk times and queues
  # [[inputs.win_perf_counters.object]]
    # Measurement = "win_disk"
    # ObjectName = "LogicalDisk"
    # Instances = ["*"]
    # Counters = [
    #   "% Idle Time",
    #   "% Disk Time",
    #   "% Disk Read Time",
    #   "% Disk Write Time",
    #   "% User Time",
    #   "% Free Space",
    #   "Current Disk Queue Length",
    #   "Free Megabytes",
    # ]

  # [[inputs.win_perf_counters.object]]
    # Measurement = "win_diskio"
    # ObjectName = "PhysicalDisk"
    # Instances = ["*"]
    # Counters = [
    #   "Disk Read Bytes/sec",
    #   "Disk Write Bytes/sec",
    #   "Current Disk Queue Length",
    #   "Disk Reads/sec",
    #   "Disk Writes/sec",
    #   "% Disk Time",
    #   "% Disk Read Time",
    #   "% Disk Write Time",
    # ]

  # [[inputs.win_perf_counters.object]]
    # Measurement = "win_net"
    # ObjectName = "Network Interface"
    # Instances = ["*"]
    # Counters = [
    # "Bytes Received/sec",
    # "Bytes Sent/sec",
    # "Packets Received/sec",
    # "Packets Sent/sec",
    # "Packets Received Discarded",
    # "Packets Outbound Discarded",
    # "Packets Received Errors",
    # "Packets Outbound Errors",
    # ]

  # [[inputs.win_perf_counters.object]]
    # Measurement = "win_system"
    # ObjectName = "System"
    # Instances = ["------"]
    # Counters = [
    #   "Context Switches/sec",
    #   "System Calls/sec",
    #   "Processor Queue Length",
    #   "System Up Time",
    # ]

  ## Example counterPath where the Instance portion must be removed to get
  ## data back, such as from the Memory object.
  # [[inputs.win_perf_counters.object]]
    # Measurement = "win_mem"
    # ObjectName = "Memory"
    ## Use 6 x - to remove the Instance bit from the counterPath.
    # Instances = ["------"]
    # Counters = [
    #   "Available Bytes",
    #   "Cache Faults/sec",
    #   "Demand Zero Faults/sec",
    #   "Page Faults/sec",
    #   "Pages/sec",
    #   "Transition Faults/sec",
    #   "Pool Nonpaged Bytes",
    #   "Pool Paged Bytes",
    #   "Standby Cache Reserve Bytes",
    #   "Standby Cache Normal Priority Bytes",
    #   "Standby Cache Core Bytes",
    # ]

  ## Example query where the Instance portion must be removed to get data back,
  ## such as from the Paging File object.
  # [[inputs.win_perf_counters.object]]
    # Measurement = "win_swap"
    # ObjectName = "Paging File"
    # Instances = ["_Total"]
    # Counters = [
    #   "% Usage",
    # ]
```

### Basics

The examples contained in this file have been found on the internet
as counters used when performance monitoring
 Active Directory and IIS in particular.
 There are a lot of other good objects to monitor, if you know what to look for.
 This file is likely to be updated in the future with more examples for
 useful configurations for separate scenarios.

For more information on concepts and terminology including object,
counter, and instance names, see the help in the Windows Performance
Monitor app.

#### Schema

*Measurement name* is specified per performance object
or `win_perf_counters` by default.

*Tags:*

- source - computer name, as specified in the `Sources` parameter. Name
          `localhost` is translated into the host name
- objectname - normalized name of the performance object
- instance - instance name, if performance object supports multiple instances,
             otherwise omitted

*Fields* are counters of the performance object.
The field name is normalized counter name.

#### Plugin wide

Plugin wide entries are underneath `[[inputs.win_perf_counters]]`.

##### PrintValid

Bool, if set to `true` will print out all matching performance objects.

Example:
`PrintValid=true`

##### UseWildcardsExpansion

If `UseWildcardsExpansion` is true, wildcards can be used in the
instance name and the counter name. Instance indexes will also be
returned in the instance name.

Partial wildcards (e.g. `chrome*`) are supported only in the instance
name on Windows Vista and newer.

If disabled, wildcards (not partial) in instance names can still be
used, but instance indexes will not be returned in the instance names.

Example:
`UseWildcardsExpansion=true`

##### LocalizeWildcardsExpansion

`LocalizeWildcardsExpansion` selects whether object and counter names
are localized when `UseWildcardsExpansion` is true and Telegraf is
running on a localized installation of Windows.

When `LocalizeWildcardsExpansion` is true, Telegraf produces metrics
with localized tags and fields even when object and counter names are
in English.

When `LocalizeWildcardsExpansion` is false, Telegraf expects object
and counter names to be in English and produces metrics with English
tags and fields.

When `LocalizeWildcardsExpansion` is false, wildcards can only be used
in instances. Object and counter names must not have wildcards.

Example:
`LocalizeWildcardsExpansion=true`

##### CountersRefreshInterval

Configured counters are matched against available counters at the interval
specified by the `CountersRefreshInterval` parameter. The default value is `1m`
(1 minute).

If wildcards are used in instance or counter names, they are expanded at this
point, if the `UseWildcardsExpansion` param is set to `true`.

Setting the `CountersRefreshInterval` too low (order of seconds) can cause
Telegraf to create a high CPU load.

Set it to `0s` to disable periodic refreshing.

Example:
`CountersRefreshInterval=1m`

##### PreVistaSupport

(Deprecated in 1.7; Necessary features on Windows Vista and newer are checked
dynamically)

Bool, if set to `true`, the plugin will use the localized PerfCounter interface
that has been present since before Vista for backwards compatibility.

It is recommended NOT to use this on OSes starting with Vista and newer because
it requires more configuration to use this than the newer interface present
since Vista.

Example for Windows Server 2003, this would be set to true:
`PreVistaSupport=true`

##### UsePerfCounterTime

Bool, if set to `true` will request a timestamp along with the PerfCounter data.
If se to `false`, current time will be used.

Supported on Windows Vista/Windows Server 2008 and newer
Example:
`UsePerfCounterTime=true`

##### IgnoredErrors

IgnoredErrors accepts a list of PDH error codes which are defined in pdh.go, if
this error is encountered it will be ignored.  For example, you can provide
"PDH_NO_DATA" to ignore performance counters with no instances, but by default
no errors are ignored.  You can find the list of possible errors here: [PDH
errors](pdh.go).

Example:
`IgnoredErrors=["PDH_NO_DATA"]`

##### Sources (Optional)

Host names or ip addresses of computers to gather all performance counters from.
The user running Telegraf must be authenticated to the remote computer(s).
E.g. via Windows sharing `net use \\SQL-SERVER-01`.
Use either localhost (`"localhost"`) or real local computer name to gather
counters also from localhost among other computers. Skip, if gather only from
localhost.

If a performance counter is present only on specific hosts set `Sources` param
on the specific counter level configuration to override global (plugin wide)
sources.

Example:
`Sources = ["localhost", "SQL-SERVER-01", "SQL-SERVER-02", "SQL-SERVER-03"]`

Default:
`Sources = ["localhost"]`

#### Object

See Entry below.

#### Entry

A new configuration entry consists of the TOML header starting with,
`[[inputs.win_perf_counters.object]]`.
This must follow before other plugin configurations,
beneath the main win_perf_counters entry, `[[inputs.win_perf_counters]]`.

Following this are 3 required key/value pairs and three optional parameters and
their usage.

##### ObjectName (Required)

ObjectName is the Object to query for, like Processor, DirectoryServices,
LogicalDisk or similar.

Example: `ObjectName = "LogicalDisk"`

##### Instances (Required)

The instances key (this is an array) declares the instances of a counter you
would like returned, it can be one or more values.

Example: `Instances = ["C:","D:","E:"]`

This will return only for the instances C:, D: and E: where relevant. To get all
instances of a Counter, use `["*"]` only.  By default any results containing
`_Total` are stripped, unless this is specified as the wanted instance.
Alternatively see the option `IncludeTotal` below.

It is also possible to set partial wildcards, eg. `["chrome*"]`, if the
`UseWildcardsExpansion` param is set to `true`

Some Objects do not have instances to select from at all.
Here only one option is valid if you want data back,
and that is to specify `Instances = ["------"]`.

##### Counters (Required)

The Counters key (this is an array) declares the counters of the ObjectName
you would like returned, it can also be one or more values.

Example: `Counters = ["% Idle Time", "% Disk Read Time", "% Disk Write Time"]`

This must be specified for every counter you want the results of, or use
`["*"]` for all the counters of the object, if the `UseWildcardsExpansion` param
is set to `true`.

##### Sources (Object) (Optional)

Overrides the [Sources](#sources-optional) global parameter for current performance
object. See [Sources](#sources-optional) description for more details.

##### Measurement (Optional)

This key is optional. If it is not set it will be `win_perf_counters`.  In
InfluxDB this is the key underneath which the returned data is stored.  So for
ordering your data in a good manner, this is a good key to set with a value when
you want your IIS and Disk results stored separately from Processor results.

Example: `Measurement = "win_disk"`

##### UseRawValues (Optional)

This key is optional. It is a simple bool.  If set to `true`, counter values
will be provided in the raw, integer, form. This is in contrast with the default
behavior, where values are returned in a formatted, displayable, form
as seen in the Windows Performance Monitor.

A field representing raw counter value has the `_Raw` suffix. Raw values should
be further used in a calculation,
e.g. `100-(non_negative_derivative("Percent_Processor_Time_Raw",1s)/100000`
Note: Time based counters (i.e. *% Processor Time*) are reported in hundredths
of nanoseconds.
This key is optional. It is a simple bool.
If set to `true`, counter values will be provided in the raw, integer, form.
This is in contrast with the default behavior, where values are returned in a
formatted, displayable, form as seen in the Windows Performance Monitor.
A field representing raw counter value has the `_Raw` suffix.
Raw values should be further used in a calculation,
e.g. `100-(non_negative_derivative("Percent_Processor_Time_Raw",1s)/100000`
Note: Time based counters (i.e. `% Processor Time`)
are reported in hundredths of nanoseconds.

Example: `UseRawValues = true`

##### IncludeTotal (Optional)

This key is optional. It is a simple bool.
If it is not set to true or included it is treated as false.
This key only has effect if the Instances key is set to `["*"]`
and you would also like all instances containing `_Total` to be returned,
like `_Total`, `0,_Total` and so on where applicable
(Processor Information is one example).

##### WarnOnMissing (Optional)

This key is optional. It is a simple bool.
If it is not set to true or included it is treated as false.
This only has effect on the first execution of the plugin.
It will print out any ObjectName/Instance/Counter combinations
asked for that do not match. Useful when debugging new configurations.

##### FailOnMissing (Internal)

This key should not be used. It is for testing purposes only.  It is a simple
bool. If it is not set to true or included this is treated as false.  If this is
set to true, the plugin will abort and end prematurely if any of the
combinations of ObjectName/Instances/Counters are invalid.

### Query examples

#### Generic Queries

```toml
[[inputs.win_perf_counters]]
  [[inputs.win_perf_counters.object]]
    # Processor usage, alternative to native, reports on a per core.
    ObjectName = "Processor"
    Instances = ["*"]
    Counters = ["% Idle Time", "% Interrupt Time", "% Privileged Time", "% User Time", "% Processor Time"]
    Measurement = "win_cpu"
    #IncludeTotal=false #Set to true to include _Total instance when querying for all (*).

  [[inputs.win_perf_counters.object]]
    # Disk times and queues
    ObjectName = "LogicalDisk"
    Instances = ["*"]
    Counters = ["% Idle Time", "% Disk Time","% Disk Read Time", "% Disk Write Time", "% User Time", "Current Disk Queue Length"]
    Measurement = "win_disk"
    #IncludeTotal=false #Set to true to include _Total instance when querying for all (*).

  [[inputs.win_perf_counters.object]]
    ObjectName = "System"
    Counters = ["Context Switches/sec","System Calls/sec", "Processor Queue Length"]
    Instances = ["------"]
    Measurement = "win_system"
    #IncludeTotal=false #Set to true to include _Total instance when querying for all (*).

  [[inputs.win_perf_counters.object]]
    # Example query where the Instance portion must be removed to get data back, such as from the Memory object.
    ObjectName = "Memory"
    Counters = ["Available Bytes","Cache Faults/sec","Demand Zero Faults/sec","Page Faults/sec","Pages/sec","Transition Faults/sec","Pool Nonpaged Bytes","Pool Paged Bytes"]
    Instances = ["------"] # Use 6 x - to remove the Instance bit from the query.
    Measurement = "win_mem"
    #IncludeTotal=false #Set to true to include _Total instance when querying for all (*).

  [[inputs.win_perf_counters.object]]
    # more counters for the Network Interface Object can be found at
    # https://msdn.microsoft.com/en-us/library/ms803962.aspx
    ObjectName = "Network Interface"
    Counters = ["Bytes Received/sec","Bytes Sent/sec","Packets Received/sec","Packets Sent/sec"]
    Instances = ["*"] # Use 6 x - to remove the Instance bit from the query.
    Measurement = "win_net"
    #IncludeTotal=false #Set to true to include _Total instance when querying for all (*).
```

#### Active Directory Domain Controller

```toml
[[inputs.win_perf_counters]]
  [inputs.win_perf_counters.tags]
    monitorgroup = "ActiveDirectory"
  [[inputs.win_perf_counters.object]]
    ObjectName = "DirectoryServices"
    Instances = ["*"]
    Counters = ["Base Searches/sec","Database adds/sec","Database deletes/sec","Database modifys/sec","Database recycles/sec","LDAP Client Sessions","LDAP Searches/sec","LDAP Writes/sec"]
    Measurement = "win_ad" # Set an alternative measurement to win_perf_counters if wanted.
    #Instances = [""] # Gathers all instances by default, specify to only gather these
    #IncludeTotal=false #Set to true to include _Total instance when querying for all (*).

  [[inputs.win_perf_counters.object]]
    ObjectName = "Security System-Wide Statistics"
    Instances = ["*"]
    Counters = ["NTLM Authentications","Kerberos Authentications","Digest Authentications"]
    Measurement = "win_ad"
    #IncludeTotal=false #Set to true to include _Total instance when querying for all (*).

  [[inputs.win_perf_counters.object]]
    ObjectName = "Database"
    Instances = ["*"]
    Counters = ["Database Cache % Hit","Database Cache Page Fault Stalls/sec","Database Cache Page Faults/sec","Database Cache Size"]
    Measurement = "win_db"
    #IncludeTotal=false #Set to true to include _Total instance when querying for all (*).
```

#### DFS Namespace + Domain Controllers

```toml
[[inputs.win_perf_counters]]
  [[inputs.win_perf_counters.object]]
    # AD, DFS N, Useful if the server hosts a DFS Namespace or is a Domain Controller
    ObjectName = "DFS Namespace Service Referrals"
    Instances = ["*"]
    Counters = ["Requests Processed","Requests Failed","Avg. Response Time"]
    Measurement = "win_dfsn"
    #IncludeTotal=false #Set to true to include _Total instance when querying for all (*).
    #WarnOnMissing = false # Print out when the performance counter is missing, either of object, counter or instance.
```

#### DFS Replication + Domain Controllers

```toml
[[inputs.win_perf_counters]]
  [[inputs.win_perf_counters.object]]
    # AD, DFS R, Useful if the server hosts a DFS Replication folder or is a Domain Controller
    ObjectName = "DFS Replication Service Volumes"
    Instances = ["*"]
    Counters = ["Data Lookups","Database Commits"]
    Measurement = "win_dfsr"
    #IncludeTotal=false #Set to true to include _Total instance when querying for all (*).
    #WarnOnMissing = false # Print out when the performance counter is missing, either of object, counter or instance.
```

#### DNS Server + Domain Controllers

```toml
[[inputs.win_perf_counters]]
  [[inputs.win_perf_counters.object]]
    ObjectName = "DNS"
    Counters = ["Dynamic Update Received","Dynamic Update Rejected","Recursive Queries","Recursive Queries Failure","Secure Update Failure","Secure Update Received","TCP Query Received","TCP Response Sent","UDP Query Received","UDP Response Sent","Total Query Received","Total Response Sent"]
    Instances = ["------"]
    Measurement = "win_dns"
    #IncludeTotal=false #Set to true to include _Total instance when querying for all (*).
```

#### IIS / ASP.NET

```toml
[[inputs.win_perf_counters]]
  [[inputs.win_perf_counters.object]]
    # HTTP Service request queues in the Kernel before being handed over to User Mode.
    ObjectName = "HTTP Service Request Queues"
    Instances = ["*"]
    Counters = ["CurrentQueueSize","RejectedRequests"]
    Measurement = "win_http_queues"
    #IncludeTotal=false #Set to true to include _Total instance when querying for all (*).

  [[inputs.win_perf_counters.object]]
    # IIS, ASP.NET Applications
    ObjectName = "ASP.NET Applications"
    Counters = ["Cache Total Entries","Cache Total Hit Ratio","Cache Total Turnover Rate","Output Cache Entries","Output Cache Hits","Output Cache Hit Ratio","Output Cache Turnover Rate","Compilations Total","Errors Total/Sec","Pipeline Instance Count","Requests Executing","Requests in Application Queue","Requests/Sec"]
    Instances = ["*"]
    Measurement = "win_aspnet_app"
    #IncludeTotal=false #Set to true to include _Total instance when querying for all (*).

  [[inputs.win_perf_counters.object]]
    # IIS, ASP.NET
    ObjectName = "ASP.NET"
    Counters = ["Application Restarts","Request Wait Time","Requests Current","Requests Queued","Requests Rejected"]
    Instances = ["*"]
    Measurement = "win_aspnet"
    #IncludeTotal=false #Set to true to include _Total instance when querying for all (*).

  [[inputs.win_perf_counters.object]]
    # IIS, Web Service
    ObjectName = "Web Service"
    Counters = ["Get Requests/sec","Post Requests/sec","Connection Attempts/sec","Current Connections","ISAPI Extension Requests/sec"]
    Instances = ["*"]
    Measurement = "win_websvc"
    #IncludeTotal=false #Set to true to include _Total instance when querying for all (*).

  [[inputs.win_perf_counters.object]]
    # Web Service Cache / IIS
    ObjectName = "Web Service Cache"
    Counters = ["URI Cache Hits %","Kernel: URI Cache Hits %","File Cache Hits %"]
    Instances = ["*"]
    Measurement = "win_websvc_cache"
    #IncludeTotal=false #Set to true to include _Total instance when querying for all (*).
```

#### Process

```toml
[[inputs.win_perf_counters]]
  [[inputs.win_perf_counters.object]]
    # Process metrics, in this case for IIS only
    ObjectName = "Process"
    Counters = ["% Processor Time","Handle Count","Private Bytes","Thread Count","Virtual Bytes","Working Set"]
    Instances = ["w3wp"]
    Measurement = "win_proc"
    #IncludeTotal=false #Set to true to include _Total instance when querying for all (*).
```

#### .NET Monitoring

```toml
[[inputs.win_perf_counters]]
  [[inputs.win_perf_counters.object]]
    # .NET CLR Exceptions, in this case for IIS only
    ObjectName = ".NET CLR Exceptions"
    Counters = ["# of Exceps Thrown / sec"]
    Instances = ["w3wp"]
    Measurement = "win_dotnet_exceptions"
    #IncludeTotal=false #Set to true to include _Total instance when querying for all (*).

  [[inputs.win_perf_counters.object]]
    # .NET CLR Jit, in this case for IIS only
    ObjectName = ".NET CLR Jit"
    Counters = ["% Time in Jit","IL Bytes Jitted / sec"]
    Instances = ["w3wp"]
    Measurement = "win_dotnet_jit"
    #IncludeTotal=false #Set to true to include _Total instance when querying for all (*).

  [[inputs.win_perf_counters.object]]
    # .NET CLR Loading, in this case for IIS only
    ObjectName = ".NET CLR Loading"
    Counters = ["% Time Loading"]
    Instances = ["w3wp"]
    Measurement = "win_dotnet_loading"
    #IncludeTotal=false #Set to true to include _Total instance when querying for all (*).

  [[inputs.win_perf_counters.object]]
    # .NET CLR LocksAndThreads, in this case for IIS only
    ObjectName = ".NET CLR LocksAndThreads"
    Counters = ["# of current logical Threads","# of current physical Threads","# of current recognized threads","# of total recognized threads","Queue Length / sec","Total # of Contentions","Current Queue Length"]
    Instances = ["w3wp"]
    Measurement = "win_dotnet_locks"
    #IncludeTotal=false #Set to true to include _Total instance when querying for all (*).

  [[inputs.win_perf_counters.object]]
    # .NET CLR Memory, in this case for IIS only
    ObjectName = ".NET CLR Memory"
    Counters = ["% Time in GC","# Bytes in all Heaps","# Gen 0 Collections","# Gen 1 Collections","# Gen 2 Collections","# Induced GC","Allocated Bytes/sec","Finalization Survivors","Gen 0 heap size","Gen 1 heap size","Gen 2 heap size","Large Object Heap size","# of Pinned Objects"]
    Instances = ["w3wp"]
    Measurement = "win_dotnet_mem"
    #IncludeTotal=false #Set to true to include _Total instance when querying for all (*).

  [[inputs.win_perf_counters.object]]
    # .NET CLR Security, in this case for IIS only
    ObjectName = ".NET CLR Security"
    Counters = ["% Time in RT checks","Stack Walk Depth","Total Runtime Checks"]
    Instances = ["w3wp"]
    Measurement = "win_dotnet_security"
    #IncludeTotal=false #Set to true to include _Total instance when querying for all (*).
```

## Troubleshooting

If you are getting an error about an invalid counter, use the `typeperf` command
to check the counter path on the command line.  E.g. `typeperf
"Process(chrome*)\% Processor Time"`

If no metrics are emitted even with the default config, you may need to repair
your performance counters.

1. Launch the Command Prompt as Administrator
   (right click "Runs As Administrator").
1. Drop into the C:\WINDOWS\System32 directory by typing `C:` then
   `cd \Windows\System32`
1. Rebuild your counter values, which may take a few moments so please be
   patient, by running:

```batchfile
lodctr /r
```

## Metrics

## Example Output
