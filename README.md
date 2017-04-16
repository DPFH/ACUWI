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
