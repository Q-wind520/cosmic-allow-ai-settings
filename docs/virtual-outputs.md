# Per-port virtual output devices

On laptops the internal speakers and the headphone jack are two ports (routes)
of a single output device. Volume panels list devices, not ports, so they cannot
switch between them.

The **Sound → Output → Separate output devices per port (experimental)** toggle
creates one virtual sink per hardware output port and loops it back into the
hardware sink. Every panel and application then lists "Speaker" and
"Headphones" as separate devices, and the hardware port follows whichever one is
the default output.

## Install

The settings toggle only writes the `virtual_outputs` key of
`com.system76.CosmicAudio`. The devices themselves are managed by a small user
service:

```sh
install -Dm0755 scripts/cosmic-virtual-outputs ~/.local/bin/cosmic-virtual-outputs
install -Dm0644 scripts/cosmic-virtual-outputs.service ~/.config/systemd/user/
systemctl --user daemon-reload
systemctl --user enable --now cosmic-virtual-outputs
```

The hardware sink is taken from `VIRTUAL_OUTPUTS_SINK`, falling back to
`alsa_output.pci-0000_04_00.6.analog-stereo`. Set the variable in the unit for a
different machine:

```ini
[Service]
Environment=VIRTUAL_OUTPUTS_SINK=alsa_output.pci-0000_00_1f.3.analog-stereo
```

## Behaviour

- One null sink plus one loopback per port of the hardware sink, with
  `latency_msec=10`.
- The hardware sink volume is held at 100% while enabled; the virtual devices
  inherit the volume that was in use, and it is restored on disable.
- The loopbacks are pinned with `sink_dont_move`/`source_dont_move` so they do
  not follow default-output changes.
- Disabling unloads the modules, restores the hardware sink as default and puts
  the previous volume and mute state back.

## Caveats

- Everything routed through the virtual devices gains the loopback latency
  (roughly 10 ms plus one graph cycle).
- The two loopback streams show up in application lists as
  "Speaker（回环）" / "Headphones（回环）". They are internal plumbing.
- The hardware sink stays visible; rename it to avoid confusion:

  ```ini
  # ~/.config/wireplumber/wireplumber.conf.d/60-analog-name.conf
  monitor.alsa.rules = [
    {
      matches = [ { node.name = "alsa_output.pci-0000_04_00.6.analog-stereo" } ]
      actions = { update-props = { node.description = "Built-in card (direct)" } }
    }
  ]
  ```

- Some third-party panels show non-ASCII descriptions as `(null)`. The
  description is present on the node; the panel needs to be fixed.
