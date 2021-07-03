# bluetooth-sync

## Configure
```bash
sudo apt -y install kodi
sudo python3 -m pip uninstall -y audio-offset-finder
sudo python3 -m pip install git+https://github.com/monotony113/audio-offset-finder.git
```

* To get the microphone: `pacmd list-sources | egrep '^\s+name: .*alsa_input'`.
* To get the output: `pacmd list-sinks | egrep '^\s+name: .*alsa_output'`.
* To get the port: look into `pacmd list-sinks` for your output.

## Run

```bash
# Desktop for testing
export MICROPHONE=alsa_input.usb-046d_0825_07B43310-02.mono-fallback
export OUTPUT=alsa_output.pci-0000_08_00.1.hdmi-stereo-extra2
export PORT=hdmi-output-2

# Laptop + projector
export MICROPHONE=alsa_input.pci-0000_00_1f.3.analog-stereo
export OUTPUT=bluez_sink.00_22_6C_0A_18_E2.a2dp_sink
export PORT=speaker-output
export CARD=bluez_card.00_22_6C_0A_18_E2

###########

# Desktop + projector
export MICROPHONE=alsa_input.usb-046d_0825_07B43310-02.mono-fallback
export OUTPUT=bluez_sink.00_22_6C_0A_18_E2.a2dp_sink
export PORT=speaker-output
export CARD=bluez_card.00_22_6C_0A_18_E2

export MONITOR="${OUTPUT}.monitor"
while true; do
  rm -f /tmp/microphone.wav /tmp/monitor.wav
  sleep 2
  timeout 30 parecord --channels=1 -d $MONITOR /tmp/monitor.wav &
  sleep 3
  timeout 24 parecord --channels=1 -d $MICROPHONE /tmp/microphone.wav
  wait
  export AUDIO_OFFSET_SEC=$(audio-offset-finder --find-offset-of /tmp/microphone.wav --within /tmp/monitor.wav | head -n1 | cut -d ' ' -f2)
  export CURRENT_LATENCY_USEC=$(pacmd list-sinks | grep "${PORT}:" | cut -d '(' -f2 | cut -d ' ' -f5)
  export REQUIRED_LATENCY_USEC=$(echo "scale=0; ((3 - ${AUDIO_OFFSET_SEC}) * 1000000) / 1" | bc)
  echo "Audio offset is ${AUDIO_OFFSET_SEC} seconds. Current latency is ${CURRENT_LATENCY_USEC}. Required latency is ${REQUIRED_LATENCY_USEC}."
  if ((REQUIRED_LATENCY_USEC >= 0 && REQUIRED_LATENCY_USEC <= 1000000)); then
    pacmd set-port-latency-offset ${CARD} ${PORT} ${REQUIRED_LATENCY_USEC}  
  else
    echo "Required latency out of range: ${REQUIRED_LATENCY_USEC}"
  fi
done
```
