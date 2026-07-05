Misc research & tools for Kali NetHunter Pro on Xiaomi MIX 2S — WiFi fix, boot images, DT patches, and notes.

---

### Device Tree Modification for WiFi Fix

To fix the WiFi issue on MIX 2S (polaris), the following property was added to the `wifi@18800000` node in `sdm845-polaris.dts`:

```dts
qcom,snoc-host-cap-skip-quirk;

Before modification:

```dts
wifi@18800000 {
    compatible = "qcom,wcn3990-wifi";
    status = "okay";
    reg = <0x00 0x18800000 0x00 0x800000>;
    reg-names = "membase";
    memory-region = <0xf4>;
    clock-names = "cxo_ref_clk_pin";
    clocks = <0x23 0x08>;
    interrupts = <0x00 0x19e 0x04 0x00 0x19f 0x04
                  0x00 0x1a0 0x04 0x00 0x1a1 0x04
                  0x00 0x1a2 0x04 0x00 0x1a3 0x04
                  0x00 0x1a4 0x04 0x00 0x1a5 0x04
                  0x00 0x1a6 0x04 0x00 0x1a7 0x04
                  0x00 0x1a8 0x04 0x00 0x1a9 0x04>;
    iommus = <0x28 0x40 0x01>;
    vdd-0.8-cx-mx-supply = <0xf5>;
    vdd-1.8-xo-supply = <0x54>;
    vdd-1.3-rfa-supply = <0x55>;
    vdd-3.3-ch0-supply = <0x56>;
    vdd-3.3-ch1-supply = <0xf6>;
};
```

After modification:

```dts
wifi@18800000 {
    compatible = "qcom,wcn3990-wifi";
    status = "okay";
    reg = <0x00 0x18800000 0x00 0x800000>;
    reg-names = "membase";
    memory-region = <0xf4>;
    clock-names = "cxo_ref_clk_pin";
    clocks = <0x23 0x08>;
    interrupts = <0x00 0x19e 0x04 0x00 0x19f 0x04
                  0x00 0x1a0 0x04 0x00 0x1a1 0x04
                  0x00 0x1a2 0x04 0x00 0x1a3 0x04
                  0x00 0x1a4 0x04 0x00 0x1a5 0x04
                  0x00 0x1a6 0x04 0x00 0x1a7 0x04
                  0x00 0x1a8 0x04 0x00 0x1a9 0x04>;
    iommus = <0x28 0x40 0x01>;
    vdd-0.8-cx-mx-supply = <0xf5>;
    vdd-1.8-xo-supply = <0x54>;
    vdd-1.3-rfa-supply = <0x55>;
    vdd-3.3-ch0-supply = <0x56>;
    vdd-3.3-ch1-supply = <0xf6>;
    qcom,snoc-host-cap-skip-quirk;   /* ← KEY FIX: skip host capability check */
};
```

---

What this quirk does

This property tells the ath10k_snoc driver to bypass the host capability request handshake with the WiFi firmware.

Before adding this quirk:

```
ath10k_snoc 18800000.wifi: msa info req rejected: 90
```

WiFi fails to initialize, wlan0 does not appear.

After adding this quirk:

```
wlan0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP
inet 192.168.2.220/24 brd 192.168.2.255 scope global dynamic wlan0
```

WiFi initializes successfully, obtains IP address via DHCP.

```