# Pulsar Wireshark Dissector

## Install dependencies

```bash
brew install protobuf glib wireshark
```

## Build examples

```bash
cmake .
make
```

## Using in macOS
1. Build Pulsar Wireshark Dissector
2. Open Wireshark
3. Click About Wireshark
4. Click Folders tab
5. See Global Plugins. etc: /Applications/Wireshark.app/Contents/PlugIns/wireshark/3-6
6. Copy `pulsar.so` to `/Applications/Wireshark.app/Contents/PlugIns/wireshark/3-6/epan`
   ```shell
      cp pulsar.so /Applications/Wireshark.app/Contents/PlugIns/wireshark/3-6/epan
   ```
7. Reopen Wireshark, you will see the Pulsar Dissector.

