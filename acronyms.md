Some of these might be old.  gleaned from docs from '83 and '86.  mostly about aegis.

abbreviations
====

acl: access control list
aclm: access control list manager
ast: active segment table
bscom: boot shell command
cal: calendar
com shell commands
ctboot: cartridge tape boot program
dm: display manager
ec: event count
flp: floppy
ftn: fortran
gmr: graphics metafile resource
gpr: graphics primitive resource
mbx: mailbox
mst: mapped segment table
netboot: diskless node bootstrap program
rfc: run file converter
rgy: registry
sau: stand-alone utilities
sh: single process shell
spm: service process manager (runs on server nodes, usually DSP*)
sysboot: system bootstrap program
uid: 64 bit identifiers for objects.
	36 creation time (since 1/1/80)
	8 bits reserved (unused?)
	20 bit node id

manager
====

NETWORK: provides remote access to the attributes and pages of existing objects
REMFILE: provides facilities to remotely create and delete objects
VTOC: maintains the volume table of contents for the disk volume.  VTOC entry (VTOCE) stores the object's attributes and provides a road map, called a file map, to the disk blocks that contain the object's pages.  used by the system to locate an object's VTOC entry given its UID.
BAT: The block availability table keeps track of the disk blocks available for allocation. The BAT manager allocates and frees disk blocks.  

