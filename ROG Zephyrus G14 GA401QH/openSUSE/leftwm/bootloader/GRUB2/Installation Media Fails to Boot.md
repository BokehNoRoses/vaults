# ISSUE:
- ## openSUSE does not contain NVIDIA propietary drivers by default (see [[Drivers]]) and will get stuck on "Hardware Detection" in the Plymouth load screen


# SOLUTION:
- ## Use \`nomodeset\` in the boot menu entry by selecting \`e\` and adding it to the \`linux\` line 

*/etc/grub2/grub.cfg*
```bash
### BEGIN /etc/grub.d/10_linux ###
{
	...
	echo 'Loading Linux 6.2.12-1-default ...'
	linux /boot/vmlinuz-6.2.12-1-default root=UUID=50f6f69f-3b76-4a0c-9ca8-941a0ef8775f ${extra_cmdline} splash=silent nomodeset resume=/dev/mapper/cr_swap mitigations=auto quiet security=apparmor nosimplefb=1
	echo    'Loading initial ramdisk ...'
	initrd  /boot/initrd-6.2.12-1-default
}
```

- ## Once drivers are installed, you can remove \`nomodeset\` from \`/etc/grub2/grub.cfg\` or \`/etc/default/grub\` (reload your config afterwards using \`sudo grub2-mkconfig -o /boot/grub2/grub.cfg\`)
