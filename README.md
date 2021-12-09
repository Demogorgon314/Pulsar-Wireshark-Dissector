# Pulsar Wireshark Dissector

**NOTE**: This is minimum support of pulsar Wireshark dissector, many commands does not support yet.

I only tested in macOS 12.1 (arm64 and x64). 

And please make sure the Wireshark version is the same as the Wireshark Lib version.

## Install Wireshark

Recommend use this way to install Wireshark in macOS.

```bash
brew install homebrew/cask/wireshark
```

## Install dependencies

```bash
brew install cmake protobuf glib wireshark
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
5. See Global Plugins. Example PATH: /Applications/Wireshark.app/Contents/PlugIns/wireshark/3-6
6. Copy `pulsar.so` to `/Applications/Wireshark.app/Contents/PlugIns/wireshark/3-6/epan`
   ```shell
      cp -f pulsar.so /Applications/Wireshark.app/Contents/PlugIns/wireshark/3-6/epan
   ```
7. Reopen Wireshark, you will see the Pulsar Dissector.

