# README.md 
This is a repo which utilizes GitHub Actions to make the `.uf2` firmware for the bluetooth micro controller for the custom split keyboard.

# Useful links
- [KeyMaps GUI](https://nickcoutsos.github.io/keymap-editor)
- [ZMK Documentation](https://zmk.dev/docs)
- [ZMK Studio](https://zmk.studio)
- [Homerow Mods](https://callistaenterprise.se/blogg/teknik/2025/01/10/homerow-mods/)

# How to pair via bluetooth
1. Check for a free bluetooth profile.
2. Select free profile(Layer 1 + [a:0,s:1,d:2,f:3,g:4]/clear profile (Layer 2 + tab)
3. Connect via bluetooth in settings
4. Validate that both halves are working
5. Profit

# LTS 
Check the GitHub Versions of this repo for stable firmware builds.

# Nice!Views
To enable nice!views correctly using the mechboards.uk corne pcb you must add the following to `config/corne.keymaps`:

```c 
// nice_view connected with control pin 8 together with LEDs
&nice_view_spi {
    cs-gpios = <&pro_micro 8 GPIO_ACTIVE_HIGH>;
};

```

