# Set gFlags using Python

Warning: this code is provided on a best effort basis and is not in any way officially supported or sanctioned by Cohesity. The code is intentionally kept simple to retain value as example code. The code in this repository is provided as-is and the author accepts no liability for damages resulting from its use.

This python script sets gflags in the morning and the evening to tune performance for backups (at night) and NAS (during the day). Not that this script is designed to run on a cluster node and will not run remotely.

## Components

* set-gflags.py: the main python script

## Deployment

* ssh into a node of the Cohesity cluster 
* Create a folder such as /home/cohesity/scripts and place the script in the folder
* Configure the settings in the script (timezone, hours to run)
* make set-gflags.py executable (chmod +x set-gflags.py)
* execute the command: crontab -e
* add the following line to crontab, then save
```text
*/10 * * * * /home/cohesity/scripts/set-gflags.py
```
This will cause the script to run every 10 minutes. BAsed on the current hour of the day, the script will apply the appropriate gflags.

The results of the script are logged in /home/cohesity/scripts/set-gflags-log.txt
```text
[cohesity@selab3-nm156s015791-node-1 scripts]$ cat set-gflags-log.txt
2019-03-04 03:06:36.630031
applying nighttime flags...
Added gflags [magneto_gatekeeper_max_tasks_per_generic_nas_entity] to the service magneto
Added gflags [bridge_magneto_nas_max_active_read_write_ops] to the service bridge
10.99.1.58 magneto_gatekeeper_max_tasks_per_generic_nas_entity set to 8
10.99.1.58 bridge_magneto_nas_max_active_read_write_ops set to 32
10.99.1.59 magneto_gatekeeper_max_tasks_per_generic_nas_entity set to 8
10.99.1.59 bridge_magneto_nas_max_active_read_write_ops set to 32
10.99.1.57 magneto_gatekeeper_max_tasks_per_generic_nas_entity set to 8
10.99.1.57 bridge_magneto_nas_max_active_read_write_ops set to 32
```
## Note about Timezones
The Cohesity cluster is set to US/Pacific time (-8). Please set my_timezone in the script to your timezone (e.g. -5 for eastern time) so that the script will treat the morning and night hours according to your local time.
