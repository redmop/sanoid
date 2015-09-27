<p align="center"><img src="http://www.openoid.net/wp-content/themes/openoid/images/sanoid_logo.png" alt="sanoid logo" title="sanoid logo"></p>
======

# Sanoid

Sanoid is a policy-driven snapshot management tool for ZFS filesystems.  When combined with the Linux KVM hypervisor, you can use it to make your systems <a href="http://openoid.net/transcend" target="_blank">functionally immortal</a>.

More prosaically, you can use Sanoid to create, automatically thin, and monitor snapshots and pool health from a single eminently human-readable TOML config file at /etc/sanoid/sanoid.conf.  (Sanoid also requires a "defaults" file located at /etc/sanoid/sanoid.defaults.conf, which is not user-editable.)  A typical Sanoid system would have a single cron job:
```
* * * * * /usr/local/bin/sanoid --cron
```

And its /etc/sanoid/sanoid.conf might look something like this:

```
[data/home]
	use_template = production
[data/images]
	use_template = production
	recursive = yes
	process_children_only = yes
[data/images/win7]
	hourly = 4

#############################
# templates below this line #
#############################

[template_production]
        hourly = 36
        daily = 30
        monthly = 3
        yearly = 0
        autosnap = yes
        autoprune = yes
```

Which would be enough to tell sanoid to take and keep 36 hourly snapshots, 30 dailies, 3 monthlies, and no yearlies for all datasets under data/images (but not data/images itself, since process_children_only is set).  Except in the case of data/images/win7-spice, which follows the same template (since it's a child of data/images) but only keeps 4 hourlies for whatever reason.

###### Sanoid Command Line Options

+ --cron

 	This will process your sanoid.conf file, create snapshots, then purge expired ones.

+ --take-snapshots

	This will process your sanoid.conf file, create snapshots, but it will NOT purge expired ones.
	
+ --prune-snapshots

	This will process your sanoid.conf file, it will NOT create snapshots, but it will purge expired ones.
	
+ --monitor-snapshots

	This option is designed to be run by a Nagios monitoring system. It reports on the health of your snapshots.
	
+ --monitor-health

	This option is designed to be run by a Nagios monitoring system. It reports on the health of the zpool your filesystems are on. It only monitors filesystems that are configured in the sanoid.conf file.
	
+ --version

	This prints the version number, and exits.

+ --verbose

	This prints additional information during the sanoid run.

+ --force-update

	This clears out sanoid's zfs snapshot listing cache. This is normally not needed.

+ --debug

	This prints out quite alot of additional information during a sanoid run, and is normally not needed.



----------

# Syncoid

Sanoid also includes a replication tool, syncoid, which facilitates the asynchronous incremental replication of ZFS filesystems.  A typical syncoid command might look like this:

```
syncoid data/images/vm backup/images/vm
```

Which would replicate the specified ZFS filesystem (aka dataset) from the data pool to the backup pool on the local system, or

```
syncoid data/images/vm root@remotehost:backup/images/vm
```

Which would push-replicate the specified ZFS filesystem from the local host to remotehost over an SSH tunnel, or

```
syncoid root@remotehost:data/images/vm backup/images/vm
```

Which would pull-replicate the filesystem from the remote host to the local system over an SSH tunnel.

Syncoid supports recursive replication (replication of a dataset and all its child datasets) and uses mbuffer buffering, lzop compression, and pv progress bars if the utilities are available on the systems used.

###### Syncoid Command Line Options

+ --[source]

	This is the source dataset. It can be either local or remote.

+ --[destination]

	This is the destination dataset. It can be either local or remote.

+ -r --recursive

	This will also transfer child datasets.

+ --compress <compression type>

	Currently accepts gzip, pigz, lzo, bzip2, pbzip2, lbzip2, lzip, plzip, xz, pxz, pixz. lzo is fast and light on the processsor and is the default, if available. pigz is fast and heavy on the processor.  bzip2, pbzip2, lbzip2, lzip, plzip, xz, pxz, and pixz should only be used on very low bandwidth connections. If the selected compression method is unavailable on the source and destination, no compression will be used.

+ --source-bwlimit <limit t|g|m|k>

	This is the bandwidth limit imposed upon the source. This is mainly used if the target does not have mbuffer installed, but bandwidth limites are desired.

+ --target-bw-limit <limit t|g|m|k>

	This is the bandwidth limit imposed upon the target. This is mainly used if the source does not have mbuffer installed, but bandwidth limites are desired.

+ --nocommandchecks

	Do not check the existance of commands before attempting the transfer. It assumes all programs are available. This should never be used.

+ --verbose

	This prints additional information during the sanoid run.

+ --debug

	This prints out quite alot of additional information during a sanoid run, and is normally not needed.

+ --dumpsnaps

	This prints a list of snapshots during the run.

+ --monitor-version

	This doesn't do anything right now.

