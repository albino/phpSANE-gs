phpSANE-gs is a web-based frontend for SANE written in HTML/PHP so you can scan with your web-browser. It also supports OCR.

This is a fork of [gawindx/phpSane](https://github.com/gawindx/phpSane). phpSANE-gs uses Ghostscript instead of Poppler to process PDFs, allowing pages to be more reliably merged (this fixes the blank pages issue) and enabling highly efficient compression of the output files.

[![Screen Shot 1](https://github.com/albino/phpSANE/blob/master/images/phpSane_Screenshot_1.png)](https://github.com/albino/phpSANE)
[![Screen Shot 2](https://github.com/albino/phpSANE/blob/master/images/phpSane_Screenshot_2.png)](https://github.com/albino/phpSANE)

## Important licensing info

This software depends on Ghostscript, which is licensed under the GNU Affero General Public License (AGPL). Because no AGPL-licensed code is distributed with phpSANE-gs, and the GPLv2 license under which the original phpSANE code was distributed is not compatible with the AGPLv3, phpSANE-gs is licensed under the GNU GPLv2. **However, if you install this software in tandem with Ghostscript, you are required to comply with the terms of the AGPL.**

What this means in practice is that if you run the software, you are responsible for making a copy of the source code available to its users. A link to this Git repository and the [Ghostscript download page](https://www.ghostscript.com/download/gsdnld.html) should suffice.

## Installing phpSANE on FreeBSD

### Prerequisites
`pkg_add -r sane-backends`  
`pkg_add -r sane-frontends`  
`pkg_add -r git*`

### Testing  
The following programs must succeed, because phpSANE is based on them.

`sane-find-scanner -q`  
`scanimage -L`  
`su -m www -c 'scanimage --test'`  

### Obtain the source code
#### Use git
`cd /var/www`  
`git clone https://github.com/gawindx/phpSane.git`

#### Or download the latest version
`mkdir /var/www/phpsane`  
`cd /var/www`  
`fetch https://github.com/gawindx/phpSane/archive/master.zip` 
`unzip master.zip`

### Set permissions
`chown -R root:www phpsane/tmp phpsane/output`  
`chmod 775 phpsane/tmp phpsane/output`

## Apache configuration
`  <Location /phpsane>`  
`    DirectoryIndex phpsane.php`  
`    Require group Admins Users Faktura`  
`   </Location>`

## OPTIONAL: 
### SELinux

If SELinux is activated and Enforced, you need to apply this custom rule :  
`semanage fcontext -a -t httpd_sys_content_t "/var/www/phpsane(/.*)?"`  
`semanage fcontext -a -t httpd_sys_rw_content_t "/var/www/phpsane/scanners(/.*)?"`  
`semanage fcontext -a -t httpd_sys_rw_content_t "/var/www/phpsane/tmp(/.*)?"`  
`semanage fcontext -a -t httpd_sys_rw_content_t "/var/www/phpsane/output(/.*)?"`  
`restorecon -R -v /var/www/phpsane`  

### devfs.rules for jails
Of course you'll ned to adjust the device name to match the USB port your scanner uses.

`pw groupadd -n scan`  
`pw groupmod scan -m www`  
`pw groupadd -n printscan`  
`pw groupmod printscan -m www`  
`service apache22 restart`

#### /etc/devfs.rules

`[devfsrules_jail_unhide_usb_printer_and_scanner=30]`  
`add include $devfsrules_hide_all`  
`add include $devfsrules_unhide_basic`  
`add include $devfsrules_unhide_login`  
`add path 'ulpt*' mode 0660 group printscan unhide`  
`add path 'unlpt*' mode 0660 group printscan unhide`  
`add path 'ugen2.8' mode 0660 group printscan unhide  # Scanner (ugen2.8 is a symlink to usb/2.8.0)`  
`add path usb unhide`  
`add path usbctl unhide`  
`add path 'usb/2.8.0' mode 0660 group printscan unhide`  

`[devfsrules_jail_unhide_usb_scanner_only=30]`  
`add include $devfsrules_hide_all`  
`add include $devfsrules_unhide_basic`  
`add include $devfsrules_unhide_login`  
`add path 'ugen2.8' mode 0660 group scan unhide  # Scanner`  
`add path usb unhide`  
`add path usbctl unhide`  
`add path 'usb/2.8.0' mode 0660 group scan unhide`  
