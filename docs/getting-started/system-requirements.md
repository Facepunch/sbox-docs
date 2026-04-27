---
title: "System Requirements"
icon: "💻"
created: 2026-04-09
updated: 2026-04-09
sources:
  - engine/Launcher/Launcher.cs
---

# System Requirements

This page outlines the hardware and software required to run the s&box Editor and player client.

## Quick API Example: Checking Capabilities

If you are developing a game and need to gracefully disable features based on the player's hardware, you can check engine capabilities at runtime:

```csharp
using Sandbox;

public sealed class HardwareCheck : Component
{
    protected override void OnStart()
    {
        // Example: Checking if the system supports VR
        if ( Sandbox.VR.VR.Enabled )
        {
            Log.Info( "VR headset detected and enabled." );
        }
        else
        {
            Log.Info( "Running in standard desktop mode." );
        }
    }
}
```

## System Requirements

| | Minimum | Recommended |
|:--|:--|:--|
| **OS** | Windows 10 (64-bit) | Windows 11 (64-bit) |
| **Processor** | Core i5-7500 / Ryzen 5 1600 | Core i7-9700K / Ryzen 7 3700 |
| **Memory** | 8 GB RAM | 16 GB RAM |
| **Graphics** | GTX 1050 / RX 570 - 4GB VRAM | RTX 2060 / RX 6600XT - 8GB VRAM |
| **Network** | Broadband Internet connection | Broadband Internet connection |
| **Storage** | 12 GB available space | 50 GB available space (SSD strongly recommended) |
| **VR Support** | OpenXR | OpenXR |

:::warning
Integrated Intel graphics are not supported. AMD integrated graphics (e.g. Ryzen APUs) will work, but performance may vary significantly depending on memory bandwidth.
:::

## Troubleshooting

:::warning Common Gotchas
- **Poor Performance on Laptops:** Ensure your laptop is plugged in and set to use the dedicated Nvidia/AMD GPU rather than integrated graphics. You can force this in your Windows Graphics Settings or Nvidia Control Panel.
- **Out of Memory Crashes:** If your project has massive, unoptimized textures and you only have 8GB of RAM, the Editor may crash during asset compilation. Try closing other memory-intensive applications (like web browsers).
- **Stuttering / Hitches:** If you are running the game off an HDD instead of an SSD, you will experience stuttering when the engine dynamically mounts addons or streams in assets over the network. An NVMe or SATA SSD is strongly recommended.
:::

- **Linux Support:** Currently, the s&box Editor is natively built for Windows. Linux players can use Proton via Steam to play games, but developer tooling on Linux is not officially supported at this time.

## Related Pages

* [Download & Install](installation.md)
* [Performance & Optimization](../systems/performance/index.md)
