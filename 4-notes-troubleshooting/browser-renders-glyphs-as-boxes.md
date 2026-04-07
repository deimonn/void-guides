# Browser renders characters as just boxes

If your installation is fresh, its likely you don't have any fonts installed to render emojis or glyphs from non-latin character sets.

Simply install a font with proper unicode coverage to fix it, for example Noto Sans:

```Shell
sudo xbps-install noto-fonts-cjk noto-fonts-emoji noto-fonts-ttf noto-fonts-ttf-extra
```
