# ACUWI
Ideas about creating updated Windows ISOs
Warning: It doesn't look like newlines are handled correctly here. Download this file in raw.

This is what you should do (in pseudocode) to create an updated windows ISO from an official one:

for each edition
single esd to separate wim
create folder
end for
for each wim
mount
get list of packages
apply updates
get list of packages
clean
unmount
end for
combine wims
remove separated wims
remove folders
create iso

This is the same thing but in commands:
dism /get-imageinfo /imagefile:install.esd
mkdir edition
dism /export-image /sourceimagefile:install.esd /sourceindex:x /destinationimagefile:edition.wim /compress:fast /checkintegrity
del install.esd
dism /mount-wim /wimfile:edition.wim /index:1 /mountdir:edition
dism /image:edition /get-packages
dism /image:edition /add-package /packagepath:..\updates\
dism /image:edition /get-packages
dism /image:edition /cleanup-image /startcomponentcleanup /resetbase 
dism /unmount-wim /mountdir:edition /commit
rmdir edition
dism /export-image /sourceimagefile:edition.wim /sourceindex:1 /Destinationimagefile:install.esd /compress:recovery /checkintegrity
oscdimg -m -oc -yoD:\bootorder.txt -bootdata:2#p0,e,b..\boot\etfsboot.com#pEF,e,b..\efi\microsoft\boot\efisys.bin -u2 -udfver102 ..\ d:\w10-x64-rc5.iso

This method lets you only use official Microsoft tools.

I'm thinking about creating a collection of scripts that will automatically do that: for this reasons, i'm saving these notes here.


-----
You can't use 64-bit Windows pE to install 32-bit Windows,
but
You can use 32-bit Windows PE to install 64-bit Windows. So we create an ISo with 32-bit windows pe and AIO install.wim.  BUT it looks like it's difficult to have an image that can boot both 64 bit and 32 bit uefi. \bootmgr.efi and \efi\microsoft\boot\efisys.bin are platform-dependent. Use these commands to create an AIO efisys.bin (needs imdisk (that is not a microsoft tool but is open source so it is acceptable) ) :
%systemroot%\system32\imdisk.exe -a -t file -f "%pe_32_64%\fwfiles\efisys.ima" -s 2880K -p "/fs:FAT /q /y" -m B:
md B:\efi\boot\
copy "%pe_32_64%\fwfiles\bootx64.efi" B:\efi\boot\
copy "%WinPERoot%\x86\Media\EFI\Boot\bootia32.efi" %pe_32_64%\media\EFI\Boot\
%systemroot%\system32\imdisk.exe -d -m B:

Thanks to reboot.pro :))
