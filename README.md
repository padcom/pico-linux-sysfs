# pico-linux-sysfs

Reading Linux's [sysfs](https://docs.kernel.org/filesystems/sysfs.html) from [Pico](https://github.com/padcom/pico-lang):
attribute files, the devices of a class (`tty`, `input`, …) and the USB device one sits on. Plain Pico over `std/fs`
(symbolic links are resolved here, not by libc), so it works interpreted too. Nothing throws: what can't be read is
`null` (or an empty list), as on a system without sysfs or in a sandbox. Every function takes the sysfs root, so tests
can run over a made-up tree.

Used by [pico-serialport](https://github.com/padcom/pico-serialport) (port listing) and
[pico-gamepad](https://github.com/padcom/pico-gamepad) (input devices). Needs Pico 0.9.0 or later.

## Use

```ts
import { classDevices, usbDeviceOf } from "padcom/pico-linux-sysfs"

for (const tty of await classDevices("tty")) {
  const device = tty.device
  if (device == null) continue                     // a virtual terminal
  const usb = await usbDeviceOf(device)
  println(`/dev/${tty.name} ${usb?.vendorId ?? ""}:${usb?.productId ?? ""} ${usb?.product ?? ""}`)
}
```

Add it with `pico add https://github.com/padcom/pico-linux-sysfs --tag v0.1.0`.

## API

- `attribute(path): Promise<string?>`: an attribute file's text, trimmed; `attributeOr(path, otherwise)`.
- `parseHex(text): int?`: `0403`, `0x1209`.
- `canonical(path): Promise<string?>`: the path with symbolic links, `.` and `..` resolved (`realpath`); `null` when
  part of it is missing.
- `classDevices(cls, sysfs = "/sys"): Promise<ClassDevice[]>`: `{ name, path, device }` for each entry of
  `/sys/class/<cls>`, in name order; `device` is the resolved `device` link (`null` for a virtual device).
- `usbDeviceOf(device, sysfs = "/sys"): Promise<UsbDevice?>`: the nearest directory at or above `device` that has an
  `idVendor`: `{ path, vendorId, productId, serialNumber?, manufacturer?, product? }`, ids as sysfs gives them.

## Tests

`pico test .` builds a small sysfs tree under `/tmp` (with the kernel's relative links) and reads it.

## License

MIT
