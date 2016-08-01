# TODOs #

1. ~~README.md instructions~~
1. ~~Turn load_config.sh into a function in an external file~~
1. ~~Remove .conf from config files~~
1. ~~Add a sample config file to be committed to master branch~~
1. ~~Detect if image file already mounted and use that~~
1. ~~Rsync options per filesystem type~~br/>
1. ~~Make sure all required variables in config file are provided or barf~~
1. ~~Remove .sh extension from all script files~~
1. Command line options
	* ~~Command line options for backup-rpi~~
		* ~~-n --dry-run~~
		* ~~-c --checksum~~
		* Probably others
	* ~~Command line options for mount-rpi script~~
		* ~~-r (readonly mount)~~
		* ~~-a (attach only - no mount)~~
		* ~~-n (dry-run - just show what would do)~~
		* ~~-f (fsck - check file systems before mounting)~~
1. ~~Integrate zerofree/sparsify options to reduce disk space (z=zerofree, s=sparsify)~~
1. Add -f option to umount-rpi so that the image can be fsck'ed just before unmounting
1. Add -f option to backup-rpi so that the image can be fsck'ed before and after
the backup process