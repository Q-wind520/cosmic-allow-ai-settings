# Switching the output port

Upstream only lists output *devices* in Settings → Sound → Output. On laptops
where the internal speakers and the headphone jack are two ports of the same
card, plugging headphones makes the speaker port unavailable, so the UI offers
no way to select it.

This fork merges the ports into the device list: a device with several routes
contributes one entry per port, labelled `port - device`, and selecting an entry
switches the port through `set_route()`.

## WirePlumber caveat

The port switch itself works, but WirePlumber drops the default output when the
active route is reported unavailable: `haveAvailableRoutes()` in
`linking-utils.lua` excludes nodes without an available active route, so
`default.audio.sink` disappears and new streams fail with "No such entity" /
"no target node available".

A one-line local patch keeps the analog sink available:

```lua
function lutils.haveAvailableRoutes (si_props, devices_om)
  -- keep analog sink selectable when its active route is unavailable
  if si_props ["node.name"] == "alsa_output.pci-0000_04_00.6.analog-stereo" then
    return true
  end
  -- ...unchanged...
end
```

Patch `/usr/share/wireplumber/scripts/lib/linking-utils.lua`, keep a backup, then
run `systemctl --user restart wireplumber`. A WirePlumber update replaces the
file, so the patch has to be re-applied afterwards.

## Command line

Without the settings app:

```sh
pactl set-sink-port alsa_output.pci-0000_04_00.6.analog-stereo analog-output-speaker
pactl set-sink-port alsa_output.pci-0000_04_00.6.analog-stereo analog-output-headphones
```
