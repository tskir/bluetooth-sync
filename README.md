# bluetooth-sync

## Configure
```bash
sudo python3 -m pip uninstall -y audio-offset-finder
sudo python3 -m pip install git+https://github.com/monotony113/audio-offset-finder.git
```

To get the microphone: `pacmd list-sources | egrep '^\s+name: .*alsa_input'`.
To get the monitor: `pacmd list-sources | egrep '^\s+name:.*\.monitor'`.

## Run

```bash
export MICROPHONE=alsa_input.usb-046d_0825_07B43310-02.mono-fallback
export MONITOR=alsa_output.pci-0000_08_00.1.hdmi-stereo-extra2.monitor

while 1; do
  rm -f /tmp/microphone.wav /tmp/monitor.wav
  sleep 2
  timeout 30 parecord --channels=1 -d $MONITOR /tmp/monitor.wav &
  sleep 3
  timeout 24 parecord --channels=1 -d $MICROPHONE /tmp/microphone.wav
  wait
  OFFSET=$(audio-offset-finder --find-offset-of /tmp/microphone.wav --within /tmp/monitor.wav | head -n1 | cut -d ' ' -f2)
  print "Determined offset to be $OFFSET"
done
```
