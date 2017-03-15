# System Configuration Access Layer (scal)

This is the **System Configuration Access Layer (scal)** for **LEDE/OpenWrt**.

It provides a **high level abstraction API** to access data models defined via
**plugins**.

Multiple plugins are permitted to provide objects belonging to the
same data model, or even extend parameters of the same objects.

This is intended to be used for implementing [TR-069](https://en.wikipedia.org/wiki/TR-069),
[NETCONF](https://tools.ietf.org/html/rfc7803) and other remote management protocols,
or even provide an abstraction for a CLI running on a router itself.

## Compiling/Installing SCAL

To compile SCAL, use these commands:

    cmake .
    make

To install:

    make install

## ubus API

This section explains the [ubus](https://lede-project.org/docs/guide-developer/ubus) API of scal.

### Object: `scald`

#### `status`

Query the status of the scal daemon.

Returns a list of data models, and for each model a list of plugins that implement it.

Example:

```json
{ "models": { "tr-181": { "plugins": [ "example" ] } } }`
```

The object will send out notifications for changes made to the data model.
The notify type describes the change type: (`"set"`, `"add"`, `"remove"`).

They contain the following fields:

- `plugin`: name of the plugin providing the object/parameter being accessed
- `model`: name of the affected data model
- `path`: path to the object
- `param`: name of the affected parameter
- `value`: new value of the affected parameter

The same events can also be received on a per-datamodel basis on the
data model object described below.

### Object: `scald.<datamodel>`

All methods that refer to an object (or an object path) take a string array
`"path"`, describing the path to the object.

Example: `[ "Device", "ManagementServer" ]` (equal to `Device.ManagementServer` in TR-181)

#### `list`

List all objects on the next level below the object specified by the `"path"` argument.
If the path is empty, the root object is listed.

Example:

```
ubus call scald.tr-181 list '{ "path": [ "Device" ] }'
```

Returns:

```json
{ "objects": [ "ManagementServer" ] }
```

#### `info`

Returns information about an object, including its list of parameters

Example:

```
ubus call scald.tr-181 info '{ "path": [ "Device", "ManagementServer" ] }'
```

Returns:

```json
{ "parameters": { "Password": { "readonly": false }, "Username": { "readonly": false } } }
```

#### `get`

Reads the value of an object parameter.
Parameter name is provided as a string in the `"name"` attribute

Example:

```
ubus call scald.tr-181 get '{ "path": [ "Device", "ManagementServer" ], "name": "Username" }'
```

Returns:

```json
{ "value": "foo" }
```

#### `set`

Sets an object parameter to a new value.

Parameter name is provided as a string in the `"name"` attribute, the value is provided as a
string in the `"value"` attribute.

Example: 

```
ubus call scald.tr-181 set '{"path": [ "Device", "ManagementServer" ], "name": "Username", "value": "baz" }'
```

#### `add`

Adds a new object instance.

Instance name is provided as a string in the `"name"` attribute.

Example:

```
ubus call scald.tr-181 add '{"path": [ "Device", "ManagementServer", "ManageableDevice" ], "name": "test" }'
```

#### `remove`

Remove an object instance.

Example:

```
ubus call scald.tr-181 remove '{"path": [ "Device", "ManagementServer", "ManageableDevice", "test" ] }'
```


### Object: `scald.acl`

This object is used to allow an external daemon to perform ACL checks for incoming requests.

After subscribing to this object, the ACL daemon receives requests as notifications.

If the ACL daemon returns a non-zero status code, the incoming request will be refused.

Example message:
  
```json
{"method":"list","plugin":"json","ubus":{"user":"root","group":"wheel"},"path":["DeviceInfo"]}
```

Plugins can add arbitrary data to this message to allow ACL filtering to be
done both before and after data model translation.

#### `method`

Name of the ubus method called on `scald.<datamodel>`.

#### `plugin`

Name of the plugin providing the object/parameter being accessed.

#### `ubus`

ACL data from the ubus client that issued the request.

#### `path`

Path to the object.

#### `param`

Name of the requested parameter.
