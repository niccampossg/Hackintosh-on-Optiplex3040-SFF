# Changelog

## v1.0 — EFI inicial (OpenCore Simplify)
- SMBIOS: iMac17,1
- macOS alvo: Monterey 12.7.6
- Framebuffer: AAPL,ig-platform-id = 00001219 (HD 530 desktop)
- Áudio: alcid=11 (Realtek ALC255/3234 — Dell OptiPlex)
- Rede: RealtekRTL8111.kext
- USB: UTBMap.kext gerado via USBToolBox (mapa próprio do gabinete)
- Quirks: AppleXcpmCfgLock=true, XhciPortLimit=false
- Porta serial desativada na BIOS (evita tela preta pós-boot)
