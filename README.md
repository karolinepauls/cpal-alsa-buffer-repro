## Commands
Request default buffer size
```
cargo run
```

Request specific fixed buffer size.
```
cargo run -- --buffer-size 1024
```

Lock device sampling rate:
```
pw-metadata -n settings 0 clock.force-quantum 2048
```

Unlock device sampling rate:
```
pw-metadata -n settings 0 clock.force-quantum 0
```

## Repro

1. Start `pw-top` in background to monitor the quanta.

2. Lock device quantum, as if another running application caused the quantum to be set (e.g. Firefox playing a video).

```
pw-metadata -n settings 0 clock.force-quantum 2048
```

3. Run, requesting default buffer size:
```
cargo run
```
Result: running with a buffer size of 512, choppy sound. Output:
```
Output device: alsa:default
Default config: SupportedStreamConfig { channels: 2, sample_rate: 48000, buffer_size: Range { min: 16, max: 262144 }, sample_format: F32 }
Requested config: StreamConfig { channels: 2, sample_rate: 48000, buffer_size: Default }
real stream buffer size Ok(512)
data len 1024 info CallbackInfo { timestamp: StreamTimestamp { callback: StreamInstant { secs: 0, nanos: 53930605 }, device: StreamInstant { secs: 0, nanos: 139263938 } }, xrun: false }
data len 1024 info CallbackInfo { timestamp: StreamTimestamp { callback: StreamInstant { secs: 0, nanos: 54107747 }, device: StreamInstant { secs: 0, nanos: 150107747 } }, xrun: false }
data len 1024 info CallbackInfo { timestamp: StreamTimestamp { callback: StreamInstant { secs: 0, nanos: 97236082 }, device: StreamInstant { secs: 0, nanos: 182569415 } }, xrun: true }
output underrun
...
```

4. Run, requesting an explicit buffer size of 512:
```
cargo run -- --buffer-size 512
```
Same effect. Output:
```
...
Requested config: StreamConfig { channels: 2, sample_rate: 48000, buffer_size: Fixed(512) }
...
```

5. Run, requesting a buffer size of at least 1024:
```
cargo run -- --buffer-size 1024
```
Result: clean sine.
```
Output device: alsa:default
Default config: SupportedStreamConfig { channels: 2, sample_rate: 48000, buffer_size: Range { min: 16, max: 262144 }, sample_format: F32 }
Requested config: StreamConfig { channels: 2, sample_rate: 48000, buffer_size: Fixed(1024) }
real stream buffer size Ok(1024)
data len 2048 info CallbackInfo { timestamp: StreamTimestamp { callback: StreamInstant { secs: 0, nanos: 53568375 }, device: StreamInstant { secs: 0, nanos: 138901708 } }, xrun: false }
data len 2048 info CallbackInfo { timestamp: StreamTimestamp { callback: StreamInstant { secs: 0, nanos: 53776726 }, device: StreamInstant { secs: 0, nanos: 160443393 } }, xrun: false }
```

6. Run with a buffer size of 512 but also ensure that pipewire runs with it.

```
pw-metadata -n settings 0 clock.force-quantum 512
```
```
cargo run -- --buffer-size 512
```
Result: clean sine.


## Observations
For some reason an app buffer size of 512 doesn't work with a device buffer size of 2048. At least with pipewire-alsa.

| app     | device   |  result |
|---------|----------|---------|
| 512     | 512      |  ok     |
| default | 2048     |  xrun   |
| 512     | 2048     |  xrun   |
| 1024    | 2048     |  ok     |
| 2048    | 2048     |  ok     |

## Versions

pipewire:
```
Compiled with libpipewire 1.6.2
Linked with libpipewire 1.6.2
```
Kubuntu 26.04.1 LTS

CPAL: git 79275c2313da30e9f8b1ad8168385d091a2c9ada (current master).
