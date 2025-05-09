<!-- markdownlint-disable MD024 -->
# Modbus Input Plugin

This plugin collects data from [Modbus][modbus] registers using e.g. Modbus TCP
or serial interfaces with Modbus RTU or Modbus ASCII.

⭐ Telegraf v1.14.0
🏷️ iot
💻 all

[modbus]: https://www.modbus.org/

## Global configuration options <!-- @/docs/includes/plugin_config.md -->

In addition to the plugin-specific configuration settings, plugins support
additional global and plugin configuration settings. These settings are used to
modify metrics, tags, and field or create aliases and configure ordering, etc.
See the [CONFIGURATION.md][CONFIGURATION.md] for more details.

[CONFIGURATION.md]: ../../../docs/CONFIGURATION.md#plugins

## Configuration

```toml @sample_general_begin.conf @sample_register.conf @sample_request.conf @sample_metric.conf @sample_general_end.conf
# Retrieve data from MODBUS slave devices
[[inputs.modbus]]
  ## Connection Configuration
  ##
  ## The plugin supports connections to PLCs via MODBUS/TCP, RTU over TCP, ASCII over TCP or
  ## via serial line communication in binary (RTU) or readable (ASCII) encoding
  ##
  ## Device name
  name = "Device"

  ## Slave ID - addresses a MODBUS device on the bus
  ## Range: 0 - 255 [0 = broadcast; 248 - 255 = reserved]
  slave_id = 1

  ## Timeout for each request
  timeout = "1s"

  ## Maximum number of retries and the time to wait between retries
  ## when a slave-device is busy.
  # busy_retries = 0
  # busy_retries_wait = "100ms"

  # TCP - connect via Modbus/TCP
  controller = "tcp://localhost:502"

  ## Serial (RS485; RS232)
  ## For RS485 specific setting check the end of the configuration.
  ## For unix-like operating systems use:
  # controller = "file:///dev/ttyUSB0"
  ## For Windows operating systems use:
  # controller = "COM1"
  # baud_rate = 9600
  # data_bits = 8
  # parity = "N"
  # stop_bits = 1

  ## Transmission mode for Modbus packets depending on the controller type.
  ## For Modbus over TCP you can choose between "TCP" , "RTUoverTCP" and
  ## "ASCIIoverTCP".
  ## For Serial controllers you can choose between "RTU" and "ASCII".
  ## By default this is set to "auto" selecting "TCP" for ModbusTCP connections
  ## and "RTU" for serial connections.
  # transmission_mode = "auto"

  ## Trace the connection to the modbus device
  # log_level = "trace"

  ## Define the configuration schema
  ##  |---register -- define fields per register type in the original style (only supports one slave ID)
  ##  |---request  -- define fields on a requests base
  ##  |---metric   -- define fields on a metric base
  configuration_type = "register"

  ## Exclude the register type tag
  ## Please note, this will also influence the grouping of metrics as you won't
  ## see one metric per register type anymore!
  # exclude_register_type_tag = false

  ## --- "register" configuration style ---

  ## Measurements
  ##

  ## Digital Variables, Discrete Inputs and Coils
  ## measurement - the (optional) measurement name, defaults to "modbus"
  ## name        - the variable name
  ## data_type   - the (optional) output type, can be BOOL or UINT16 (default)
  ## address     - variable address

  discrete_inputs = [
    { name = "start",          address = [0]},
    { name = "stop",           address = [1]},
    { name = "reset",          address = [2]},
    { name = "emergency_stop", address = [3]},
  ]
  coils = [
    { name = "motor1_run",     address = [0]},
    { name = "motor1_jog",     address = [1]},
    { name = "motor1_stop",    address = [2]},
  ]

  ## Analog Variables, Input Registers and Holding Registers
  ## measurement - the (optional) measurement name, defaults to "modbus"
  ## name        - the variable name
  ## byte_order  - the ordering of bytes
  ##  |---AB, ABCD   - Big Endian
  ##  |---BA, DCBA   - Little Endian
  ##  |---BADC       - Mid-Big Endian
  ##  |---CDAB       - Mid-Little Endian
  ## data_type   - BIT (single bit of a register)
  ##               INT8L, INT8H, UINT8L, UINT8H (low and high byte variants)
  ##               INT16, UINT16, INT32, UINT32, INT64, UINT64,
  ##               FLOAT16-IEEE, FLOAT32-IEEE, FLOAT64-IEEE (IEEE 754 binary representation)
  ##               FIXED, UFIXED (fixed-point representation on input)
  ##               STRING (byte-sequence converted to string)
  ## bit         - (optional) bit of the register, ONLY valid for BIT type
  ## scale       - the final numeric variable representation
  ## address     - variable address

  holding_registers = [
    { name = "power_factor", byte_order = "AB",   data_type = "FIXED", scale=0.01,  address = [8]},
    { name = "voltage",      byte_order = "AB",   data_type = "FIXED", scale=0.1,   address = [0]},
    { name = "energy",       byte_order = "ABCD", data_type = "FIXED", scale=0.001, address = [5,6]},
    { name = "current",      byte_order = "ABCD", data_type = "FIXED", scale=0.001, address = [1,2]},
    { name = "frequency",    byte_order = "AB",   data_type = "UFIXED", scale=0.1,  address = [7]},
    { name = "power",        byte_order = "ABCD", data_type = "UFIXED", scale=0.1,  address = [3,4]},
    { name = "firmware",     byte_order = "AB",   data_type = "STRING", address = [5, 6, 7, 8, 9, 10, 11, 12]},
  ]
  input_registers = [
    { name = "tank_level",   byte_order = "AB",   data_type = "INT16",   scale=1.0,     address = [0]},
    { name = "tank_ph",      byte_order = "AB",   data_type = "INT16",   scale=1.0,     address = [1]},
    { name = "pump1_speed",  byte_order = "ABCD", data_type = "INT32",   scale=1.0,     address = [3,4]},
  ]

  ## --- "request" configuration style ---

  ## Per request definition
  ##

  ## Define a request sent to the device
  ## Multiple of those requests can be defined. Data will be collated into metrics at the end of data collection.
  [[inputs.modbus.request]]
    ## ID of the modbus slave device to query.
    ## If you need to query multiple slave-devices, create several "request" definitions.
    slave_id = 1

    ## Byte order of the data.
    ##  |---ABCD -- Big Endian (Motorola)
    ##  |---DCBA -- Little Endian (Intel)
    ##  |---BADC -- Big Endian with byte swap
    ##  |---CDAB -- Little Endian with byte swap
    byte_order = "ABCD"

    ## Type of the register for the request
    ## Can be "coil", "discrete", "holding" or "input"
    register = "coil"

    ## Name of the measurement.
    ## Can be overridden by the individual field definitions. Defaults to "modbus"
    # measurement = "modbus"

    ## Request optimization algorithm.
    ##  |---none       -- Do not perform any optimization and use the given layout(default)
    ##  |---shrink     -- Shrink requests to actually requested fields
    ##  |                 by stripping leading and trailing omits
    ##  |---rearrange  -- Rearrange request boundaries within consecutive address ranges
    ##  |                 to reduce the number of requested registers by keeping
    ##  |                 the number of requests.
    ##  |---max_insert -- Rearrange request keeping the number of extra fields below the value
    ##                    provided in "optimization_max_register_fill". It is not necessary to define 'omitted'
    ##                    fields as the optimisation will add such field only where needed.
    # optimization = "none"

    ## Maximum number register the optimizer is allowed to insert between two fields to
    ## save requests.
    ## This option is only used for the 'max_insert' optimization strategy.
    ## NOTE: All omitted fields are ignored, so this option denotes the effective hole
    ## size to fill.
    # optimization_max_register_fill = 50

    ## Field definitions
    ## Analog Variables, Input Registers and Holding Registers
    ## address        - address of the register to query. For coil and discrete inputs this is the bit address.
    ## name *1        - field name
    ## type *1,2      - type of the modbus field, can be
    ##                  BIT (single bit of a register)
    ##                  INT8L, INT8H, UINT8L, UINT8H (low and high byte variants)
    ##                  INT16, UINT16, INT32, UINT32, INT64, UINT64 and
    ##                  FLOAT16, FLOAT32, FLOAT64 (IEEE 754 binary representation)
    ##                  STRING (byte-sequence converted to string)
    ## length *1,2    - (optional) number of registers, ONLY valid for STRING type
    ## bit *1,2       - (optional) bit of the register, ONLY valid for BIT type
    ## scale *1,2,4   - (optional) factor to scale the variable with
    ## output *1,3,4  - (optional) type of resulting field, can be INT64, UINT64 or FLOAT64.
    ##                  Defaults to FLOAT64 for numeric fields if "scale" is provided.
    ##                  Otherwise the input "type" class is used (e.g. INT* -> INT64).
    ## measurement *1 - (optional) measurement name, defaults to the setting of the request
    ## omit           - (optional) omit this field. Useful to leave out single values when querying many registers
    ##                  with a single request. Defaults to "false".
    ##
    ## *1: These fields are ignored if field is omitted ("omit"=true)
    ## *2: These fields are ignored for both "coil" and "discrete"-input type of registers.
    ## *3: This field can only be "UINT16" or "BOOL" if specified for both "coil"
    ##     and "discrete"-input type of registers. By default the fields are
    ##     output as zero or one in UINT16 format unless "BOOL" is used.
    ## *4: These fields cannot be used with "STRING"-type fields.

    ## Coil / discrete input example
    fields = [
      { address=0, name="motor1_run" },
      { address=1, name="jog", measurement="motor" },
      { address=2, name="motor1_stop", omit=true },
      { address=3, name="motor1_overheating", output="BOOL" },
      { address=4, name="firmware", type="STRING", length=8 },
    ]

    [inputs.modbus.request.tags]
      machine = "impresser"
      location = "main building"

  [[inputs.modbus.request]]
    ## Holding example
    ## All of those examples will result in FLOAT64 field outputs
    slave_id = 1
    byte_order = "DCBA"
    register = "holding"
    fields = [
      { address=0, name="voltage",      type="INT16",   scale=0.1   },
      { address=1, name="current",      type="INT32",   scale=0.001 },
      { address=3, name="power",        type="UINT32",  omit=true   },
      { address=5, name="energy",       type="FLOAT32", scale=0.001, measurement="W" },
      { address=7, name="frequency",    type="UINT32",  scale=0.1   },
      { address=8, name="power_factor", type="INT64",   scale=0.01  },
    ]

    [inputs.modbus.request.tags]
      machine = "impresser"
      location = "main building"

  [[inputs.modbus.request]]
    ## Input example with type conversions
    slave_id = 1
    byte_order = "ABCD"
    register = "input"
    fields = [
      { address=0, name="rpm",         type="INT16"                   },  # will result in INT64 field
      { address=1, name="temperature", type="INT16", scale=0.1        },  # will result in FLOAT64 field
      { address=2, name="force",       type="INT32", output="FLOAT64" },  # will result in FLOAT64 field
      { address=4, name="hours",       type="UINT32"                  },  # will result in UIN64 field
    ]

    [inputs.modbus.request.tags]
      machine = "impresser"
      location = "main building"

  ## --- "metric" configuration style ---

  ## Per metric definition
  ##

  ## Request optimization algorithm across metrics
  ##  |---none       -- Do not perform any optimization and just group requests
  ##  |                 within metrics (default)
  ##  |---max_insert -- Collate registers across all defined metrics and fill in
  ##                    holes to optimize the number of requests.
  # optimization = "none"

  ## Maximum number of registers the optimizer is allowed to insert between
  ## non-consecutive registers to save requests.
  ## This option is only used for the 'max_insert' optimization strategy and
  ## effectively denotes the hole size between registers to fill.
  # optimization_max_register_fill = 50

  ## Define a metric produced by the requests to the device
  ## Multiple of those metrics can be defined. The referenced registers will
  ## be collated into requests send to the device
  [[inputs.modbus.metric]]
    ## ID of the modbus slave device to query
    ## If you need to query multiple slave-devices, create several "metric" definitions.
    slave_id = 1

    ## Byte order of the data
    ##  |---ABCD -- Big Endian (Motorola)
    ##  |---DCBA -- Little Endian (Intel)
    ##  |---BADC -- Big Endian with byte swap
    ##  |---CDAB -- Little Endian with byte swap
    # byte_order = "ABCD"

    ## Name of the measurement
    # measurement = "modbus"

    ## Field definitions
    ## register    - type of the modbus register, can be "coil", "discrete",
    ##               "holding" or "input". Defaults to "holding".
    ## address     - address of the register to query. For coil and discrete inputs this is the bit address.
    ## name        - field name
    ## type *1     - type of the modbus field, can be
    ##                 BIT (single bit of a register)
    ##                 INT8L, INT8H, UINT8L, UINT8H (low and high byte variants)
    ##                 INT16, UINT16, INT32, UINT32, INT64, UINT64 and
    ##                 FLOAT16, FLOAT32, FLOAT64 (IEEE 754 binary representation)
    ##                 STRING (byte-sequence converted to string)
    ## length *1   - (optional) number of registers, ONLY valid for STRING type
    ## bit *1,2    - (optional) bit of the register, ONLY valid for BIT type
    ## scale *1,3  - (optional) factor to scale the variable with
    ## output *2,3 - (optional) type of resulting field, can be INT64, UINT64 or FLOAT64. Defaults to FLOAT64 if
    ##               "scale" is provided and to the input "type" class otherwise (i.e. INT* -> INT64, etc).
    ##
    ## *1: These fields are ignored for both "coil" and "discrete"-input type of registers.
    ## *2: This field can only be "UINT16" or "BOOL" if specified for both "coil"
    ##     and "discrete"-input type of registers. By default the fields are
    ##     output as zero or one in UINT16 format unless "BOOL" is used.
    ## *3: These fields cannot be used with "STRING"-type fields.
    fields = [
      { register="coil",    address=0, name="door_open"},
      { register="coil",    address=1, name="status_ok"},
      { register="holding", address=0, name="voltage",      type="INT16"   },
      { address=1, name="current",      type="INT32",   scale=0.001 },
      { address=5, name="energy",       type="FLOAT32", scale=0.001 },
      { address=7, name="frequency",    type="UINT32",  scale=0.1   },
      { address=8, name="power_factor", type="INT64",   scale=0.01  },
      { address=9, name="firmware",     type="STRING",  length=8    },
    ]

    ## Tags assigned to the metric
    # [inputs.modbus.metric.tags]
    #   machine = "impresser"
    #   location = "main building"

  ## RS485 specific settings. Only take effect for serial controllers.
  ## Note: This has to be at the end of the modbus configuration due to
  ## TOML constraints.
  # [inputs.modbus.rs485]
    ## Delay RTS prior to sending
    # delay_rts_before_send = "0ms"
    ## Delay RTS after to sending
    # delay_rts_after_send = "0ms"
    ## Pull RTS line to high during sending
    # rts_high_during_send = false
    ## Pull RTS line to high after sending
    # rts_high_after_send = false
    ## Enabling receiving (Rx) during transmission (Tx)
    # rx_during_tx = false

  ## Enable workarounds required by some devices to work correctly
  # [inputs.modbus.workarounds]
    ## Pause after connect delays the first request by the specified time.
    ## This might be necessary for (slow) devices.
    # pause_after_connect = "0ms"

    ## Pause between read requests sent to the device.
    ## This might be necessary for (slow) serial devices.
    # pause_between_requests = "0ms"

    ## Close the connection after every gather cycle.
    ## Usually the plugin closes the connection after a certain idle-timeout,
    ## however, if you query a device with limited simultaneous connectivity
    ## (e.g. serial devices) from multiple instances you might want to only
    ## stay connected during gather and disconnect afterwards.
    # close_connection_after_gather = false

    ## Force the plugin to read each field in a separate request.
    ## This might be necessary for devices not conforming to the spec,
    ## see https://github.com/influxdata/telegraf/issues/12071.
    # one_request_per_field = false

    ## Enforce the starting address to be zero for the first request on
    ## coil registers. This is necessary for some devices see
    ## https://github.com/influxdata/telegraf/issues/8905
    # read_coils_starting_at_zero = false

    ## String byte-location in registers AFTER byte-order conversion
    ## Some device (e.g. EM340) place the string byte in only the upper or
    ## lower byte location of a register see
    ## https://github.com/influxdata/telegraf/issues/14748
    ## Available settings:
    ##   lower -- use only lower byte of the register i.e. 00XX 00XX 00XX 00XX
    ##   upper -- use only upper byte of the register i.e. XX00 XX00 XX00 XX00
    ## By default both bytes of the register are used i.e. XXXX XXXX.
    # string_register_location = ""
```

## Notes

You can debug Modbus connection issues by enabling `debug_connection`. To see
those debug messages, Telegraf has to be started with debugging enabled
(i.e. with the `--debug` option). Please be aware that connection tracing will
produce a lot of messages and should __NOT__ be used in production environments.

Please use `pause_after_connect` / `pause_between_requests` with care. Ensure
the total gather time, including the pause(s), does not exceed the configured
collection interval. Note that pauses add up if multiple requests are sent!

## Configuration styles

The modbus plugin supports multiple configuration styles that can be set using
the `configuration_type` setting. The different styles are described
below. Please note that styles cannot be mixed, i.e. only the settings belonging
to the configured `configuration_type` are used for constructing _modbus_
requests and creation of metrics.

Directly jump to the styles:

- [original / register plugin style](#register-configuration-style)
- [per-request style](#request-configuration-style)
- [per-metrict style](#metric-configuration-style)

---

### `register` configuration style

This is the original style used by this plugin. It allows a per-register
configuration for a single slave-device.

> [!NOTE]
> For legacy reasons this configuration style is not completely consistent with the other styles.

#### Usage of `data_type`

The field `data_type` defines the representation of the data value on input from
the modbus registers.  The input values are then converted from the given
`data_type` to a type that is appropriate when sending the value to the output
plugin. These output types are usually an integer or floating-point-number. The
size of the output type is assumed to be large enough for all supported input
types. The mapping from the input type to the output type is fixed and cannot
be configured.

##### Booleans: `BOOL`

This type is only valid for _coil_ and _discrete_ registers. The value will be
`true` if the register has a non-zero (ON) value and `false` otherwise.

##### Integers: `INT8L`, `INT8H`, `UINT8L`, `UINT8H`

These types are used for 8-bit integer values. Select the one that matches your
modbus data source. The `L` and `H` suffix denotes the low- and high byte of
the register respectively.

##### Integers: `INT16`, `UINT16`, `INT32`, `UINT32`, `INT64`, `UINT64`

These types are used for integer input values. Select the one that matches your
modbus data source. For _coil_ and _discrete_ registers only `UINT16` is valid.

##### Floating Point: `FLOAT16-IEEE`, `FLOAT32-IEEE`, `FLOAT64-IEEE`

Use these types if your modbus registers contain a value that is encoded in this
format. These types always include the sign, therefore no variant exists.

##### Fixed Point: `FIXED`, `UFIXED`

These types are handled as an integer type on input, but are converted to
floating point representation for further processing (e.g. scaling). Use one of
these types when the input value is a decimal fixed point representation of a
non-integer value.

Select the type `UFIXED` when the input type is declared to hold unsigned
integer values, which cannot be negative. The documentation of your modbus
device should indicate this by a term like 'uint16 containing fixed-point
representation with N decimal places'.

Select the type `FIXED` when the input type is declared to hold signed integer
values. Your documentation of the modbus device should indicate this with a term
like 'int32 containing fixed-point representation with N decimal places'.

##### String: `STRING`

This type is used to query the number of registers specified in the `address`
setting and convert the byte-sequence to a string. Please note, if the
byte-sequence contains a `null` byte, the string is truncated at this position.
You cannot use the `scale` setting for string fields.

##### Bit: `BIT`

This type is used to query a single bit of a register specified in the `address`
setting and convert the value to an unsigned integer. This type __requires__ the
`bit` setting to be specified.

---

### `request` configuration style

This style can be used to specify the modbus requests directly. It enables
specifying multiple `[[inputs.modbus.request]]` sections including multiple
slave-devices. This way, _modbus_ gateway devices can be queried. Please note
that _requests_ might be split for non-consecutive addresses. If you want to
avoid this behavior please add _fields_ with the `omit` flag set filling the
gaps between addresses.

#### Slave device

You can use the `slave_id` setting to specify the ID of the slave device to
query. It should be specified for each request, otherwise it defaults to
zero. Please note, only one `slave_id` can be specified per request.

#### Byte order of the register

The `byte_order` setting specifies the byte and word-order of the registers. It
can be set to `ABCD` for _big endian (Motorola)_ or `DCBA` for _little endian
(Intel)_ format as well as `BADC` and `CDAB` for _big endian_ or _little endian_
with _byte swap_.

#### Register type

The `register` setting specifies the modbus register-set to query and can be set
to `coil`, `discrete`, `holding` or `input`.

#### Per-request measurement setting

You can specify the name of the measurement for the following field definitions
using the `measurement` setting. If the setting is omitted `modbus` is
used. Furthermore, the measurement value can be overridden by each field
individually.

#### Optimization setting

__Please only use request optimization if you do understand the implications!__
The `optimization` setting can be used to optimize the actual requests sent to
the device. The following algorithms are available

##### `none` (_default_)

Do not perform any optimization. Please note that the requests are still obeying
the maximum request sizes. Furthermore, completely empty requests, i.e. all
fields specify `omit=true`, are removed. Otherwise, the requests are sent as
specified by the user including request of omitted fields. This setting should
be used if you want full control over the requests e.g. to accommodate for
device constraints.

##### `shrink`

This optimization allows to remove leading and trailing fields from requests if
those fields are omitted. This can shrink the request number and sizes in cases
where you specify large amounts of omitted fields, e.g. for documentation
purposes.

##### `rearrange`

Requests are processed similar to `shrink` but the request boundaries are
rearranged such that usually less registers are being read while keeping the
number of requests. This optimization algorithm only works on consecutive
address ranges and respects user-defined gaps in the field addresses.

__Please note:__ This optimization might take long in case of many
non-consecutive, non-omitted fields!

##### `aggressive`

Requests are processed similar to `rearrange` but user-defined gaps in the field
addresses are filled automatically. This usually reduces the number of requests,
but will increase the number of registers read due to larger requests.
This algorithm might be useful if you only want to specify the fields you are
interested in but want to minimize the number of requests sent to the device.

__Please note:__ This optimization might take long in case of many
non-consecutive, non-omitted fields!

##### `max_insert`

Fields are assigned to the same request as long as the hole between the fields
do not exceed the maximum fill size given in `optimization_max_register_fill`.
User-defined omitted fields are ignored and interpreted as holes, so the best
practice is to not manually insert omitted fields for this optimizer. This
allows to specify only actually used fields and let the optimizer figure out
the request organization which can dramatically improve query time. The
trade-off here is between the cost of reading additional registers trashed
later and the cost of many requests.

__Please note:__ The optimal value for `optimization_max_register_fill` depends
on the network and the queried device. It is hence recommended to test several
values and assess performance in order to find the best value. Use the
`--test --debug` flags to monitor how may requests are sent and the number of
touched registers.

#### Field definitions

Each `request` can contain a list of fields to collect from the modbus device.

##### address

A field is identified by an `address` that reflects the modbus register
address. You can usually find the address values for the different data-points
in the datasheet of your modbus device. This is a mandatory setting.

For _coil_ and _discrete input_ registers this setting specifies the __bit__
containing the value of the field.

##### name

Using the `name` setting you can specify the field-name in the metric as output
by the plugin. This setting is ignored if the field's `omit` is set to `true`
and can be omitted in this case.

__Please note:__ There cannot be multiple fields with the same `name` in one
metric identified by `measurement`, `slave_id` and `register`.

##### register datatype

The `type` setting specifies the datatype of the modbus register and can be
set to `INT8L`, `INT8H`, `UINT8L`, `UINT8H` where `L` is the lower byte of the
register and `H` is the higher byte.
Furthermore, the types `INT16`, `UINT16`, `INT32`, `UINT32`, `INT64` or `UINT64`
for integer types or `FLOAT16`, `FLOAT32` and `FLOAT64` for IEEE 754 binary
representations of floating point values exist. `FLOAT16` denotes a
half-precision float with a 16-bit representation.
Usually the datatype of the register is listed in the datasheet of your modbus
device in relation to the `address` described above.

The `STRING` datatype is special in that it requires the `length` setting to
be specified containing the length (in terms of number of registers) containing
the string. The returned byte-sequence is interpreted as string and truncated
to the first `null` byte found if any. The `scale` and `output` setting cannot
be used for this `type`.

This setting is ignored if the field's `omit` is set to `true` or if the
`register` type is a bit-type (`coil` or `discrete`) and can be omitted in
these cases.

##### scaling

You can use the `scale` setting to scale the register values, e.g. if the
register contains a fix-point values in `UINT32` format with two decimal places
for example. To convert the read register value to the actual value you can set
the `scale=0.01`. The scale is used as a factor e.g. `field_value * scale`.

This setting is ignored if the field's `omit` is set to `true` or if the
`register` type is a bit-type (`coil` or `discrete`) and can be omitted in these
cases.

__Please note:__ The resulting field-type will be set to `FLOAT64` if no output
format is specified.

##### output datatype

Using the `output` setting you can explicitly specify the output
field-datatype. The `output` type can be `INT64`, `UINT64` or `FLOAT64`. If not
set explicitly, the output type is guessed as follows: If `scale` is set to a
non-zero value, the output type is `FLOAT64`. Otherwise, the output type
corresponds to the register datatype _class_, i.e. `INT*` will result in
`INT64`, `UINT*` in `UINT64` and `FLOAT*` in `FLOAT64`.

This setting is ignored if the field's `omit` is set to `true` and can be
omitted. In case the `register` type is a bit-type (`coil` or `discrete`) only
`UINT16` or `BOOL` are valid with the former being the default if omitted.
For `coil` and `discrete` registers the field-value is output as zero or one in
`UINT16` format or as `true` and `false` in `BOOL` format.

#### per-field measurement setting

The `measurement` setting can be used to override the measurement name on a
per-field basis. This might be useful if you want to split the fields in one
request to multiple measurements. If not specified, the value specified in the
[`request` section](#per-request-measurement-setting) or, if also omitted,
`modbus` is used.

This setting is ignored if the field's `omit` is set to `true` and can be
omitted in this case.

#### omitting a field

When specifying `omit=true`, the corresponding field will be ignored when
collecting the metric but is taken into account when constructing the modbus
requests. This way, you can fill "holes" in the addresses to construct
consecutive address ranges resulting in a single request. Using a single modbus
request can be beneficial as the values are all collected at the same point in
time.

#### Tags definitions

Each `request` can be accompanied by tags valid for this request.

__Please note:__ These tags take precedence over predefined tags such as `name`,
`type` or `slave_id`.

---

### `metric` configuration style

This style can be used to specify the desired metrics directly instead of
focusing on the modbus view. Multiple `[[inputs.modbus.metric]]` sections
including multiple slave-devices can be specified. This way, _modbus_ gateway
devices can be queried. The plugin automatically collects registers across
the specified metrics, groups them per slave and register-type and (optionally)
optimizes the resulting requests for non-consecutive addresses.

#### Slave device

You can use the `slave_id` setting to specify the ID of the slave device to
query. It should be specified for each metric section, otherwise it defaults to
zero. Please note, only one `slave_id` can be specified per metric section.

#### Byte order of the registers

The `byte_order` setting specifies the byte and word-order of the registers. It
can be set to `ABCD` for _big endian (Motorola)_ or `DCBA` for _little endian
(Intel)_ format as well as `BADC` and `CDAB` for _big endian_ or _little endian_
with _byte swap_.

#### Measurement name

You can specify the name of the measurement for the fields defined in the
given section using the `measurement` setting. If the setting is omitted
`modbus` is used.

#### Optimization setting

__Please only use request optimization if you do understand the implications!__
The `optimization` setting can specified globally, i.e. __NOT__ per metric
section, and is used to optimize the actual requests sent to the device. Here,
the optimization is applied across _all metric sections_! The following
algorithms are available

##### `none` (_default_)

Do not perform any optimization. Please note that consecutive registers are
still grouped into one requests while obeying the maximum request sizes. This
setting should be used if you want to touch as less registers as possible at
the cost of more requests sent to the device.

##### `max_insert`

Fields are assigned to the same request as long as the hole between the touched
registers does not exceed the maximum fill size given via
`optimization_max_register_fill`. This optimization might lead to a drastically
reduced request number and thus an improved query time. The trade-off here is
between the cost of reading additional registers trashed later and the cost of
many requests.

__Please note:__ The optimal value for `optimization_max_register_fill` depends
on the network and the queried device. It is hence recommended to test several
values and assess performance in order to find the best value. Use the
`--test --debug` flags to monitor how may requests are sent and the number of
touched registers.

#### Field definitions

Each `metric` can contain a list of fields to collect from the modbus device.
The specified fields directly corresponds to the fields of the resulting metric.

##### register

The `register` setting specifies the modbus register-set to query and can be set
to `coil`, `discrete`, `holding` or `input`.

##### address

A field is identified by an `address` that reflects the modbus register
address. You can usually find the address values for the different data-points
in the datasheet of your modbus device. This is a mandatory setting.

For _coil_ and _discrete input_ registers this setting specifies the __bit__
containing the value of the field.

##### name

Using the `name` setting you can specify the field-name in the metric as output
by the plugin.

__Please note:__ There cannot be multiple fields with the same `name` in one
metric identified by `measurement`, `slave_id`, `register` and tag-set.

##### register datatype

The `type` setting specifies the datatype of the modbus register and can be
set to `INT8L`, `INT8H`, `UINT8L`, `UINT8H` where `L` is the lower byte of the
register and `H` is the higher byte.
Furthermore, the types `INT16`, `UINT16`, `INT32`, `UINT32`, `INT64` or `UINT64`
for integer types or `FLOAT16`, `FLOAT32` and `FLOAT64` for IEEE 754 binary
representations of floating point values exist. `FLOAT16` denotes a
half-precision float with a 16-bit representation.
Usually the datatype of the register is listed in the datasheet of your modbus
device in relation to the `address` described above.

The `STRING` datatype is special in that it requires the `length` setting to
be specified containing the length (in terms of number of registers) containing
the string. The returned byte-sequence is interpreted as string and truncated
to the first `null` byte found if any. The `scale` and `output` setting cannot
be used for this `type`.

This setting is ignored if the `register` is a bit-type (`coil` or `discrete`)
and can be omitted in these cases.

##### scaling

You can use the `scale` setting to scale the register values, e.g. if the
register contains a fix-point values in `UINT32` format with two decimal places
for example. To convert the read register value to the actual value you can set
the `scale=0.01`. The scale is used as a factor e.g. `field_value * scale`.

This setting is ignored if the `register` is a bit-type (`coil` or `discrete`)
and can be omitted in these cases.

__Please note:__ The resulting field-type will be set to `FLOAT64` if no output
format is specified.

##### output datatype

Using the `output` setting you can explicitly specify the output
field-datatype. The `output` type can be `INT64`, `UINT64` or `FLOAT64`. If not
set explicitly, the output type is guessed as follows: If `scale` is set to a
non-zero value, the output type is `FLOAT64`. Otherwise, the output type
corresponds to the register datatype _class_, i.e. `INT*` will result in
`INT64`, `UINT*` in `UINT64` and `FLOAT*` in `FLOAT64`.

In case the `register` is a bit-type (`coil` or `discrete`) only `UINT16` or
`BOOL` are valid with the former being the default if omitted. For `coil` and
`discrete` registers the field-value is output as zero or one in `UINT16` format
or as `true` and `false` in `BOOL` format.

#### Tags definitions

Each `metric` can be accompanied by a set of tag. These tags directly correspond
to the tags of the resulting metric.

__Please note:__ These tags take precedence over predefined tags such as `name`,
`type` or `slave_id`.

---

## Troubleshooting

### Strange data

Modbus documentation is often a mess. People confuse memory-address (starts at
one) and register address (starts at zero) or are unsure about the word-order
used. Furthermore, there are some non-standard implementations that also swap
the bytes within the register word (16-bit).

If you get an error or don't get the expected values from your device, you can
try the following steps (assuming a 32-bit value).

If you are using a serial device and get a `permission denied` error, check the
permissions of your serial device and change them accordingly.

In case you get an `exception '2' (illegal data address)` error you might try to
offset your `address` entries by minus one as it is very likely that there is
confusion between memory and register addresses.

If you see strange values, the `byte_order` might be wrong. You can either probe
all combinations (`ABCD`, `CDBA`, `BADC` or `DCBA`) or set `byte_order="ABCD"
data_type="UINT32"` and use the resulting value(s) in an online converter like
[this][online-converter]. This especially makes sense if you don't want to mess
with the device, deal with 64-bit values and/or don't know the `data_type` of
your register (e.g. fix-point floating values vs. IEEE floating point).

If your data still looks corrupted, please post your configuration, error
message and/or the output of `byte_order="ABCD" data_type="UINT32"` to one of
the telegraf support channels (forum, slack or as an issue).  If nothing helps,
please post your configuration, error message and/or the output of
`byte_order="ABCD" data_type="UINT32"` to one of the telegraf support channels
(forum, slack or as an issue).

[online-converter]: https://www.scadacore.com/tools/programming-calculators/online-hex-converter/

### Workarounds

Some Modbus devices need special read characteristics when reading data and will
fail otherwise. For example, some serial devices need a pause between register
read requests. Others might only support a limited number of simultaneously
connected devices, like serial devices or some ModbusTCP devices. In case you
need to access those devices in parallel you might want to disconnect
immediately after the plugin finishes reading.

To enable this plugin to also handle those "special" devices, there is the
`workarounds` configuration option. In case your documentation states certain
read requirements or you get read timeouts or other read errors, you might want
to try one or more workaround options.  If you find that other/more workarounds
are required for your device, please let us know.

In case your device needs a workaround that is not yet implemented, please open
an issue or submit a pull-request.

## Metrics

The plugin reads the configured registers and constructs metrics based on the
specified configuration. There is no predefined metric format.

## Example Output

```text
modbus,name=device,slave_id=1,type=holding_register energy=3254.5,power=23.5,frequency=49,97 1701777274026591864
```
