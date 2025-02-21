# machineid provides support for reading the unique machine id of most host OS's (without admin privileges)

![Image of Gopher 47](logo.png)

… because sometimes you just need to reliably identify your machines.

## Fork

[This](https://github.com/fourcorelabs/machineid) is a fork of [github.com/panta/machineid](https://github.com/panta/machineid).

## Main Features

* Cross-Platform (tested on Win7+, Debian 8+, Ubuntu 14.04+, OS X 10.6+, FreeBSD 11+)
* No admin privileges required
* Hardware independent (no usage of MAC, BIOS or CPU — those are too unreliable, especially in a VM environment)
* IDs are unique<sup>[1](#unique-key-reliability)</sup> to the installed OS

## Installation

Get the library with

```bash
go get github.com/fourcorelabs/machineid
```

You can also add the cli app directly to your `$GOPATH/bin` with

```bash
go get github.com/fourcorelabs/machineid/cmd/machineid
```

## Usage

```golang
package main

import (
  "fmt"
  "log"
  "github.com/fourcorelabs/machineid"
)

func main() {
  id, err := machineid.ID()
  if err != nil {
    log.Fatal(err)
  }
  fmt.Println(id)
}
```

Or even better, use securely hashed machine IDs:

```golang
package main

import (
  "fmt"
  "log"
  "github.com/fourcorelabs/machineid"
)

func main() {
  id, err := machineid.ProtectedID("myAppName")
  if err != nil {
    log.Fatal(err)
  }
  fmt.Println(id)
}
```

### Function: ID() (string, error)

Returns original machine id as a `string`.

### Function: ProtectedID(appID string) (string, error)

Returns hashed version of the machine ID as a `string`. The hash is generated in a cryptographically secure way, using a fixed, application-specific key (calculates HMAC-SHA256 of the app ID, keyed by the machine ID).

## What you get

This package returns the OS native machine UUID/GUID, which the OS uses for internal needs.

All machine IDs are usually generated during system installation and stay constant for all subsequent boots.

The following sources are used:

* **BSD** uses `/etc/hostid` and `smbios.system.uuid` as a fallback
* **Linux** uses `$MACHINE_ID_FILE` (if not empty), `/var/lib/dbus/machine-id`, `/etc/machine-id` ([man](http://man7.org/linux/man-pages/man5/machine-id.5.html))
* **OS X** uses `IOPlatformUUID`
* **Windows** uses the `MachineGuid` from `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Cryptography`

## Unique Key Reliability

Do note, that `machine-id` and `MachineGuid` can be changed by root/admin, although that may not come without cost (broken system services and more).
Most IDs won't be regenerated by the OS, when you clone/image/restore a particular OS installation. This is a well known issue with cloned windows installs (not using the official sysprep tools).

**Linux** users can generate a new id with `dbus-uuidgen` and put the id into `/var/lib/dbus/machine-id` and `/etc/machine-id`.
**Windows** users can use the `sysprep` [toolchain](https://docs.microsoft.com/en-us/windows-hardware/manufacture/desktop/sysprep--generalize--a-windows-installation) to create images, which produce valid images ready for distribution. Such images produce a new unique machine ID on each deployment.

## Security Considerations

A machine ID uniquely identifies the host. Therefore it should be considered "confidential", and must not be exposed in untrusted environments. If you need a stable unique identifier for your app, do not use the machine ID directly.

> A reliable solution is to hash the machine ID in a cryptographically secure way, using a fixed, application-specific key.

That way the ID will be properly unique, and derived in a constant way from the machine ID but there will be no way to retrieve the original machine ID from the application-specific one.

Do something along these lines:

```golang
package main

import (
  "crypto/hmac"
  "crypto/sha256"
  "fmt"
  "github.com/fourcorelabs/machineid"
)

const appKey = "WowSuchNiceApp"

func main() {
  id, _ := machineid.ID()
  fmt.Println(protect(appKey, id))
  // Output: dbabdb7baa54845f9bec96e2e8a87be2d01794c66fdebac3df7edd857f3d9f97
}

func protect(appID, id string) string {
  mac := hmac.New(sha256.New, []byte(id))
  mac.Write([]byte(appID))
  return fmt.Sprintf("%x", mac.Sum(nil))
}
```

Or simply use the convenience API call:

```golang
hashedID, err := machineid.ProtectedID("myAppName")
```

## Snippets

Don't want to download code, and just need a way to get the data by yourself?

BSD:

```bash
cat /etc/hostid
# or (might be empty)
kenv -q smbios.system.uuid
```

Linux:

```bash
cat /var/lib/dbus/machine-id
# or when not found (e.g. Fedora 20)
cat /etc/machine-id
```

OS X:

```bash
ioreg -rd1 -c IOPlatformExpertDevice | grep IOPlatformUUID
```

Windows:

```batch
reg query HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Cryptography /v MachineGuid
```
or
* Open Windows Registry via `regedit`
* Navigate to `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Cryptography`
* Take value of key `MachineGuid`

## Credits

The [original library](github.com/denisbrodbeck/machineid) was created by [Denis Brodbeck](https://github.com/denisbrodbeck).
The Go gopher was created by [Denis Brodbeck](https://github.com/denisbrodbeck) with [gopherize.me](https://gopherize.me/), based on original artwork from [Renee French](http://reneefrench.blogspot.com/).

## License

The MIT License (MIT) — [Denis Brodbeck](https://github.com/denisbrodbeck). Please have a look at the [LICENSE.md](LICENSE.md) for more details.
