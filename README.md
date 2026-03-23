# Complete *arr Stack Setup Guide for Unraid

A comprehensive guide to setting up and configuring a complete media automation stack on Unraid, including Sonarr, Radarr, Jackett, Jellyseerr, Jellyfin, and advanced quality control.

![arr Stack](https://img.shields.io/badge/Platform-Unraid-orange)
![License](https://img.shields.io/badge/License-MIT-blue)
![Maintenance](https://img.shields.io/badge/Maintained-Yes-green)

## 📖 Table of Contents

- [Overview](#overview)
- [What's Included](#whats-included)
- [Quick Start](#quick-start)
- [Documentation](#documentation)
- [Features](#features)
- [Prerequisites](#prerequisites)
- [Support](#support)
- [Contributing](#contributing)
- [License](#license)

## 🎯 Overview

This repository contains comprehensive documentation for setting up a fully automated media management system on Unraid. The stack handles everything from content discovery and downloading to organization and streaming, with advanced quality control to ensure you only get the best releases.

**Two Setup Options:**
- **Local Downloads**: Transmission with VPN on your Unraid server
- **Remote Seedbox**: Using a seedbox (Ultra.cc recommended) with Syncthing for automatic file transfers

## 📦 What's Included

### Core Applications

- **[Sonarr](https://sonarr.tv/)** - TV show management and automation
- **[Radarr](https://radarr.video/)** - Movie management and automation
- **[Jackett](https://github.com/Jackett/Jackett)** - Indexer proxy for torrent trackers
- **[Jellyseerr](https://github.com/Fallenbagel/jellyseerr)** - Request management with beautiful UI
- **[Profilarr](https://github.com/Dictionarry-Hub/profilarr)** - Profile and custom format sync manager
- **[Jellyfin](https://jellyfin.org/)** - Open-source media server

### Download Clients

- **[Transmission](https://transmissionbt.com/)** - Lightweight torrent client
- **[Syncthing](https://syncthing.net/)** - Continuous file synchronization (for seedbox setups)

### Supporting Tools

- **TRaSH Guides Integration** - Community-maintained quality profiles and custom formats
- **Remote Path Mapping** - Seamless seedbox integration
- **Hardlink Support** - Space-efficient file management

## 🚀 Quick Start

1. **Read the Prerequisites** - Ensure you have Unraid set up with Community Applications
2. **Choose Your Setup** - Local downloads or remote seedbox
3. **Follow the Setup Guide** - [Complete Setup Guide](docs/setup-guide.md)
4. **Configure Quality Profiles** - [Quality Profiles Guide](docs/quality-profiles-guide.md)
5. **Test and Enjoy** - Start requesting content!

## 📚 Documentation

### Setup Guides

- **[Complete Setup Guide](docs/setup-guide.md)** - Full walkthrough of installing and configuring the entire stack
  - Folder structure planning
  - Application installation (step-by-step)
  - Seedbox setup with Syncthing
  - Remote path mapping configuration
  - Integration between all components

- **[Quality Profiles Guide](docs/quality-profiles-guide.md)** - Advanced guide to filtering and quality control
  - Understanding quality profiles
  - Custom formats and scoring
  - Blocking bad releases
  - TRaSH Guides integration
  - Testing and troubleshooting

### Additional Documentation

- **[Troubleshooting Guide](docs/troubleshooting.md)** - Common issues and solutions
- **[Seedbox Configuration Examples](docs/seedbox-examples.md)** - Real-world setup examples
- **[Maintenance Guide](docs/maintenance.md)** - Keeping your stack running smoothly

## ✨ Features

### Automated Media Management
- 🎬 Automatic TV show and movie downloads
- 📺 Episode and season tracking
- 🔄 Automatic upgrades to better quality
- 📝 Intelligent renaming and organization
- 🎯 Request management for users

### Quality Control
- 🎚️ Size-based filtering (MB/minute limits)
- 🏷️ Custom format scoring system
- 🚫 Bad release group blocking
- ✅ Preferred codec and audio support
- 📊 TRaSH Guides integration

### Seedbox Support
- 🌍 Remote downloading in another country
- 🔒 Keep torrenting off your home IP
- ⚡ Fast seedbox speeds (Ultra.cc recommended)
- 🔄 Automatic Syncthing transfers
- 🗺️ Remote path mapping configuration

### User Experience
- 🎨 Beautiful request interface (Jellyseerr)
- 👥 Multi-user support with permissions
- 📱 Mobile apps for Jellyfin
- 🔔 Notifications (Discord, Email, Telegram)
- 🎮 One-click request and watch

## 📋 Prerequisites

- Unraid server (6.9.0 or later recommended)
- Community Applications plugin installed
- Sufficient storage space (depends on your media collection)
- Basic understanding of Docker containers
- **Choose one:**
  - VPN subscription (for local downloads)
  - Seedbox service (Ultra.cc recommended for remote downloads)

## 🎓 Recommended Learning Path

1. **Start Here**: Read the [Complete Setup Guide](docs/setup-guide.md)
   - Install all applications
   - Configure basic settings
   - Get content downloading

2. **Quality Control**: Read the [Quality Profiles Guide](docs/quality-profiles-guide.md)
   - Set up quality gates
   - Filter bad releases
   - Optimize for your preferences

3. **Fine-Tuning**: Reference the [Troubleshooting Guide](docs/troubleshooting.md)
   - Solve common issues
   - Optimize performance
   - Customize to your needs

## 🛠️ Technology Stack

```
┌─────────────────────────────────────────────┐
│              Jellyseerr (Requests)          │
└──────────────┬──────────────────────────────┘
               │
      ┌────────┴────────┐
      │                 │
┌─────▼─────┐    ┌─────▼─────┐
│   Sonarr  │    │   Radarr  │
│   (TV)    │    │  (Movies) │
└─────┬─────┘    └─────┬─────┘
      │                │
      └────────┬────────┘
               │
        ┌──────▼──────┐
        │   Jackett   │
        │  (Indexers) │
        └──────┬──────┘
               │
    ┌──────────┴──────────┐
    │                     │
┌───▼────┐         ┌──────▼───────┐
│Transmis│         │   Seedbox    │
│ -sion  │         │ + Syncthing  │
│ (VPN)  │         │  (Ultra.cc)  │
└───┬────┘         └──────┬───────┘
    │                     │
    └──────────┬──────────┘
               │
        ┌──────▼──────┐
        │   Jellyfin  │
        │   (Stream)  │
        └─────────────┘
```

## 💡 Why This Stack?

### Open Source
All core components are free and open-source. No license fees, no tracking, complete control.

### Automated
Set it up once, then it runs itself. Request content and it appears in your library automatically.

### Quality Focused
Advanced filtering ensures you get the best quality releases, not bloated files or bad encodes.

### Privacy Options
Use a seedbox to keep torrenting completely off your home network and IP address.

### Community Driven
Built on years of community knowledge, TRaSH Guides, and real-world testing.

## 📊 Default Ports Reference

| Application | Port | Purpose |
|------------|------|---------|
| Transmission | 9091 | Web UI (local downloads) |
| Syncthing | 8384 | Web UI (seedbox sync) |
| Jackett | 9117 | Indexer proxy |
| Sonarr | 8989 | TV management |
| Radarr | 7878 | Movie management |
| Jellyseerr | 5055 | Request management |
| Profilarr | 5500 | Profile sync |
| Jellyfin | 8096 | Media streaming |

## 🤝 Support

- **Issues**: Found a problem? [Open an issue](https://github.com/johnpernock/arr-stack/issues)
- **Discussions**: Questions or ideas? [Start a discussion](https://github.com/johnpernock/arr-stack/discussions)
- **Unraid Forums**: [r/unraid](https://reddit.com/r/unraid)
- **Application Support**:
  - [Sonarr Discord](https://discord.gg/sonarr)
  - [Radarr Discord](https://discord.gg/radarr)
  - [Jellyfin Forum](https://forum.jellyfin.org/)

## 🌟 Acknowledgments

- **[TRaSH Guides](https://trash-guides.info/)** - Comprehensive quality profiles and custom formats
- **[LinuxServer.io](https://www.linuxserver.io/)** - Excellent Docker containers
- **[Unraid](https://unraid.net/)** - The platform that makes this all easy
- **[Ultra.cc](https://ultra.cc/)** - Recommended seedbox provider
- The entire open-source community maintaining these amazing tools

## 📜 License

This documentation is released under the MIT License. See [LICENSE](LICENSE) for details.

The applications referenced in this guide are owned by their respective maintainers and released under their own licenses.

## 🔄 Contributing

Contributions are welcome! If you have improvements, corrections, or additional tips:

1. Fork this repository
2. Create a feature branch (`git checkout -b feature/improvement`)
3. Commit your changes (`git commit -am 'Add new configuration tip'`)
4. Push to the branch (`git push origin feature/improvement`)
5. Open a Pull Request

---

**⭐ If you found this guide helpful, please star the repository!**

**📖 Ready to get started? Head over to the [Complete Setup Guide](docs/setup-guide.md)**
