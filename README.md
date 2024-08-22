# Zigpio

Zig library for Raspberry Pi GPIO.

This library contains direct bindings to the popular [pigpio](https://github.com/joan2937/pigpio) library.

## Requirements
- Raspberry Pi
- Zig compiler installed on your local machine
- Pigpio shared library installed at `/lib/libpigpio.so.1` on the Raspberry Pi. This is the default location for the pigpio library on Raspbian.

## Installation

You'll be cloning this repo into your project's root directory, next to your `build.zig`.

```
root
┗ build.zig
┗ src/
  ┗ main.zig
```
Create a `libs` directory and clone Zigpio into it.

```bash
mkdir libs
cd ./libs
git clone https://github.com/erik-dunteman/zigpio.git
```

Your project should now look like this:
```
root
┗ build.zig
┗ libs/
  ┗ zigpio/
    ┗ src/
      ┗ zigpio.zig
      ┗ ...
┗ src/
  ┗ main.zig
```


Modify your `build.zig` to include Zigpio in your build.

Note since we're building for a Raspberry Pi, we'll be using specific target options to cross-compile for the Raspberry Pi. This build.zig has been verified for a Raspberry Pi Zero W running Headless Rasberry Pi OS.

### Build.zig
```zig
const std = @import("std");

pub fn build(b: *std.Build) void {
    // Target options for a Raspberry Pi Zero W running Headless Raspberry Pi OS
    const target = b.resolveTargetQuery(.{
        .abi = .gnueabihf,
        .cpu_arch = .arm,
        .os_tag = .linux,
        .cpu_model = std.Target.Query.CpuModel{ .explicit = &std.Target.arm.cpu.arm1176jz_s },
    });
    const optimize = b.standardOptimizeOption(.{});

    // Add your own project files
    const exe = b.addExecutable(.{
        .name = "zigblink", // Name of your executable
        .root_source_file = b.path("src/main.zig"),
        .target = target,
        .optimize = optimize,
    });

    // Include the zigpio library as a module
    // This allows you to @import("zigpio") in your project
    const zigpio = b.addModule(
        "zigpio",
        .{
            .root_source_file = b.path("libs/zigpio/src/zigpio.zig"),
        },
    );
    exe.root_module.addImport("zigpio", zigpio);

    // We need to link against libc for the dynamic linking to work
    exe.linkLibC();

    // Run the build
    b.installArtifact(exe);
}
```

### Building

Run your build:
```bash
zig build --summary all
```

And this will produce a `zigblink` executable in your `zig-out` directory.

You'll then need to transfer the `zigblink` executable to your Raspberry Pi, using a tool such as [scp](https://www.geeksforgeeks.org/scp-command-in-linux-with-examples/).

You're also free to build on the Raspberry Pi itself, though you may find it to be slower than building on your local machine.

### Running
Pigpio requires root privileges to run, so you'll need to run with `sudo`.
```bash
sudo ./zigblink
```

---

# Docs

