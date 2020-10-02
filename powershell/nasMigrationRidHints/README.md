# Get NAS Migration Rid Hints using PowerShell

Warning: this code is provided on a best effort basis and is not in any way officially supported or sanctioned by Cohesity. The code is intentionally kept simple to retain value as example code. The code in this repository is provided as-is and the author accepts no liability for damages resulting from its use.

This script will gather the incremental replication details for views recovered/migrated from protected NAS volumes/mount points, and generate the needed iris_cli commands to enable incremental first-run replication for the recovered view.

The assumption is that NAS volumes have been protected to cluster A and replicated to cluster B. Then the NAS volume is recovered to a view, and the new view has been protected to cluster A with replication to cluster B. We can then use the last replication task ID of the NAS volume as a hint for the first replication of the view, by setting a gflag. This causes the view replication to be incremental rather than full.

## Download the script

Run these commands from PowerShell to download the script(s) into your current directory

```powershell
# Download Commands
$scriptName = 'nasMigrationRidHints'
$repoURL = 'https://raw.githubusercontent.com/bseltz-cohesity/scripts/master/powershell'
(Invoke-WebRequest -Uri "$repoUrl/$scriptName/$scriptName.ps1").content | Out-File "$scriptName.ps1"; (Get-Content "$scriptName.ps1") | Set-Content "$scriptName.ps1"
(Invoke-WebRequest -Uri "$repoUrl/cohesity-api/cohesity-api.ps1").content | Out-File cohesity-api.ps1; (Get-Content cohesity-api.ps1) | Set-Content cohesity-api.ps1
# End Download Commands
```

## Components

* nasMigrationRidHints: the main powershell script
* cohesity-api.ps1: the Cohesity REST API helper module

Place both files in a folder together, then we can run the script.

First, run the script WITHOUT the -commit switch to see what snapshots would be modified. The parameters used below tell the script to check the past 14 days of snapshots (for all jobs) to see if any need adjusting. No changes will be made without the -commit parameter.

```powershell
./nasMigrationRidHints.ps1 -vip mycluster `
                           -username myuser `
                           -domain mydomain.net `
                           -daysAgo 1 `
                           -remoteCluster othercluster
```

The output will look something like this:

```text
Connected!

--------

View Protection Job: ntap7-vol0-backups
View Name: ntap7-vol0
NAS Job ID: 137404
NAS Entity: \\netapp7.seltzer.net\vol0
Replication ID: 49940811

iris_cli cluster update-gflag gflag-name=bridge_madrox_ancestor_rid_hint gflag-value="dummy:dummy:1,ntap7-vol0-backups:ntap7-vol0:49940811" reason="madrox seed" effective-now=true service-name=bridge

--------

View Protection Job: ntap7-scripts-backup
View Name: ntap7-scripts
NAS Job ID: 137404
NAS Entity: \\netapp7.seltzer.net\scripts
Replication ID: 49940812

iris_cli cluster update-gflag gflag-name=bridge_madrox_ancestor_rid_hint gflag-value="dummy:dummy:1,ntap7-scripts-backup:ntap7-scripts:49940812" reason="madrox seed" effective-now=true service-name=bridge

--------

Clean Up Gflag:

iris_cli cluster update-gflag gflag-name=bridge_madrox_ancestor_rid_hint clear=true reason="madrox seed" effective-now=true service-name=bridge
```

You can then apply the gflag change and perform the initial replication of the protected view, which will be incremental (0 bytes should be transferred during the initial replication).

## Parameters

* -vip: Cohesity Cluster to connect to
* -username: Cohesity username
* -domain: (optional) Active Directory domain of user (defaults to local)
* -daysAgo: (optional) number of days back to search for recoveries (default is 1)
* -remoteCluster: name of replica cluster