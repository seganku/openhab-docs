---
layout: documentation
title: JsonDB Storage
---

# JsonDB Storage

JsonDB provides a system database for storage of configuration data.
All configuration data stored into the system through the REST interface that is used by the user interfaces will be stored into the JsonDB.
JsonDB provides a high performance, human readable data store that is primarily meant for system use but can be edited manually, or stored in a version control system such as Git.

## Technical Overview

The system stores different data into separate tables.
JsonDB maps these tables into separate files - in this way each file contains a different type of data (eg. Things, Items, Links).
The system also keeps a number of backups in a ```backup``` folder.
Each time a file is updated, the current version will be moved to the ```backup``` folder and timestamped so that the system can retain the most recent files.
By default the last 5 copies of each file are retained.
When the system loads data from the file system, should it find that a file is corrupted it will attempt to open the most recent backup - it will try each backup in turn until a file is correctly read, or the number of files is reached.

To improve performance and reduce disk use all file writes are deferred for a few hundred milliseconds.
This ensures that if there are multiple updates of the database in a short time, the system will only write these updates to the file system after the group of updates completes.
If the system gets into a loop such that it is continually updating configuration information in the database, JsonDB will write a file every minute.
These timers can be configured by the user along with the number of backup files retained.

It is worth noting that data is only read from the file system when the table is first created - this is normally on system startup.
After this the data is retained in memory and only written to file when there are changes.

## Manual Editing

Data is stored in a "pretty" format to make it more human readable, and is sorted so ordering is not random (important when a version control system is used).
It is therefore editable by advanced users who might want to do a search and replace on item names etc.

If you manually edit the file you must take responsibility for ensuring it is correctly formatted.
A Json format checker (such as jsonlint.com) can be used to check the format and this should ensure that the file can be correctly read.
It doesn't however ensure that the correct format is used - users wanting to edit a specific table are advised to first configure the system with the UI and then use the format generated by the UI as a template for subsequent additions and changes.
Most data stored in the database is written in a way that should be understandable by someone with good knowledge of the system.

As stated above, the files are only read during system startup - therefore if you change a file you will need to stop openHAB, make your changes and restart the system for the changes to take effect.

---

openHAB stores configuration information in JSON (JavaScript Object Notation) formatted (structured) text files located in the `OPENHAB_USERDATA/jsondb/` directory.

## Storage Scope

All configuration information regarding _**Items, Links, and Things**_ are defined via the User Interfaces (UI, REST) or via internal openHAB services.

::: tip Note
The JSON DB does NOT store information for manually configured _**Items, Links, or Things**_, since these are already stored in files within the `OPENHAB_CONF` sub-directories (e.g. `/etc/openhab/items/`).
:::

## Storage Purpose

JSON DB Storage can be used for:

- Backup: Maintains a copy of your configurations in the `OPENHAB_USERDATA/jsondb/backup` directory
- Troubleshooting: Allows the user to interact with the configurations that were automatically generated via the UIs
- Advanced administration: Gives the possibility to manually define configuration parameters within the `*.json` files

## Storage Use

openHAB writes the `*.json` files every time a change is made via the User Interfaces.
openHAB _**reads the `*.json` files only once at startup**_.  So, if you edit them manually, you should restart openHAB.

The system employs two write mechanisms to improve performance where there are multiple writes in a short time. When the service is closed, all open services are written to disk.
The parameters for the two mechanisms may be modified in UI :arrow_right: Settings :arrow_right: Json Storage

1. _Write delay_ (defaults to 500 ms): Sets the time to wait before writing changes to disk.
  This can reduce the number of writes when many changes are being introduced within a short period, and
1. _Maximum write delay_ (defaults to 30000 ms): Sets the maximum period the service will wait to write data in cases where changes are happening continually.

The service keeps up to five backups of each table.
The outdated file is copied to the backup folder and then that file is overwritten with the new content.

## Storage Location

The JsonDB Storage resides in the `OPENHAB_USERDATA/jsondb/` directory.
The full directory path depends on the installation method:

- Linux Repository Installation: `/var/lib/openhab/jsondb/`
- Linux Manual Installation: `/opt/openhab/userdata/jsondb/`
- Windows (Manual) Installation: `C:\openHAB\userdata\jsondb\`

Within the `OPENHAB_USERDATA/jsondb/` directory, you will find the following files:

| Filename                                                        | _Contents_                            |
|-----------------------------------------------------------------|---------------------------------------|
| org.openhab.config.discovery.**DiscoveryResult.json** | _Results of UI Discovery_       |
| org.openhab.core.items.**Item.json**                  | _Items configurations_                |
| org.openhab.core.thing.link.**ItemChannelLink.json**  | _Item to Channel Link configurations_ |
| org.openhab.core.thing.link.**ItemThingLink.json**    | _Item to Thing Link configurations_   |
| org.openhab.core.thing.**Thing.json**                 | _Things configurations_               |

## Example Use

In this example, we will use the Network Binding (2.0) to Search for Things, add a new Thing to openHAB and then modify its parameters to check the information that is stored in the JsonDB.

Step 1. Add new Thing (name: `ISP_Gateway`) from UI:

![Add_Thing_UI](./images/ui_add_thing.png)

Step 2. Check the contents of the `OPENHAB_USERDATA/jsondb/org.openhab.core.thing.Thing.json` file:

```json
root@rpi3:~# more /var/lib/openhab/jsondb/org.openhab.core.thing.Thing.json
{
  "network:device:172_16_13_254": {
    "class": "org.openhab.core.thing.internal.ThingImpl",
    "value": {
      "label": "ISP_Gateway",
      "channels": [
        {
          "acceptedItemType": "Switch",
          "kind": "STATE",
          "uid": {
            "segments": [
              "network",
              "device",
              "172_16_13_254",
              "online"
            ]
          },
          "channelTypeUID": {
            "segments": [
              "network",
              "online"
            ]
          },
          "configuration": {
            "properties": {}
          },
          "properties": {},
          "defaultTags": []
        },
        {
          "acceptedItemType": "Number",
          "kind": "STATE",
          "uid": {
          "segments": [
              "network",
              "device",
              "172_16_13_254",
              "time"
            ]
          },
          "channelTypeUID": {
            "segments": [
              "network",
              "time"
            ]
          },
          "configuration": {
            "properties": {}
          },
          "properties": {},
          "defaultTags": []
        }
      ],
      "configuration": {
        "properties": {
          "hostname": "172.16.13.254",
          "refresh_interval": 60000,
          "port": 0,
          "dhcplisten": false,
          "retry": 1,
          "timeout": 5000,
          "use_system_ping": false
        }
      },
      "properties": {},
      "uid": {
          "segments": [
          "network",
          "device",
          "172_16_13_254"
        ]
      },
      "thingTypeUID": {
        "segments": [
          "network",
          "device"
        ]
      }
    }
  }
}
```

Step 3. Using UI :arrow_right: Settings :arrow_right: Things, edit the new `ISP_Gateway` Thing and modify the following parameters:

- Location (from unset to `MyHome`)
- Retry (from 1 to 3)
- Timeout (from 5000 to 10000)

and save:

![Edit_Thing_UI](./images/ui_edit_thing.png)

Step 4. Check the configuration properties again in the `OPENHAB_USERDATA/jsondb/org.openhab.core.thing.Thing.json` file:
![New_Json](./images/new_json_file.png)
