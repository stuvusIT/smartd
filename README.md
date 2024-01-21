# smartd

This role installs and configures `smartd`.
In the default configuration, all devices will be tested and observed, but no mails will be sent.

## Requirements

Debian, Ubuntu, Arch Linux, or Fedora
`/usr/bin/mail` has to exist and be configured to be able to send report mails if that is desired.

## Role Variables

This role only uses one top-level var called `smartd_devices`, which is a dict of dicts to specify device monitoring.
Each key corresponds to one drive path, while `DEVICESCAN` (the only default entry) may be used to scan for devices.
After a `DEVICESCAN` entry, all further lines are ignored by smartd, so you probably want to insert it at the end or not at all.

All possible options inside the dict are briefly described here.
You can find a detailed wall-of-text for each one on the [smartd man page](https://www.smartmontools.org/browser/trunk/smartmontools/smartd.conf.5.in).

| Name                         |                Mandatory / Default                | Description                                                                                                                                                                       |
|:-----------------------------|:-------------------------------------------------:|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `type`                       |                      `auto`                       | The type of device, e.g. `ata`, `scsi`, `marvell`, etc.                                                                                                                           |
| `nocheck`                    |                  `standby,15,q`                   | Powermode setting. By default, devices in `standby` are only woken up every 15th attempt and skipped check attempts are not logged.                                               |
| `check_type`                 |                     `normal`                      | Use `normal` or `permissive` to force SMART checking even if it is not advertised.                                                                                                |
| `ata_offline_testing`        |                                                   | Set to `on` or `off` to enable or disable automatic ATA offline testing by the device itself. Note that you have to quote the value because of ansible's interpretation of `off`. |
| `attribute_autosave`         |                      `true`                       | Enables or disables attribute autosave on startup                                                                                                                                 |
| `check_health_return_status` |                      `true`                       | Check device health status using the SMART return status                                                                                                                          |
| `report_error_types`         | `[error,xerror,selftest,offlinests,"scterc,0,0"]` | List of error types to report. By default, SMART errors as well as failed tests are logged and TLER will be disabled.                                                             |
| `non_smart_settings`         |                `["lookahead,on"]`                 | List of non-SMART options to set                                                                                                                                                  |
| `test_schedule`              |   <code>(L/../../6/01&#124;S/../.././02)</code>   | REGEXP to specify self-test schedules. By default, a short test is executed daily after 1am. A long test is executed each sunday after 2am.                                       |
| `mail_recipients`            |                       `[]]`                       | List of mail addresses to report to                                                                                                                                               |
| `mail_frequency`             |                   `diminishing`                   | Frequency of report emails. `once`, `daily` and `diminishing` are possible values.                                                                                                |
| `mail_script`                |                                                   | Path to a script that will be executed in addition to sending report mails                                                                                                        |
| `report_ata_failure`         |                      `true`                       | Report failure of any ATA usage attributes                                                                                                                                        |
| `report_ata_prefail`         |                      `true`                       | Report change of any ATA prefail attributes                                                                                                                                       |
| `report_ata_usage`           |                      `false`                      | Report any time a usage attribute has changed                                                                                                                                     |
| `ignore_ata_failure_ids`     |                       `[]`                        | List of SMART IDs to ignore when checking for failures                                                                                                                            |
| `ignore_ata_usage_ids`       |                       `[]`                        | List of SMART IDs to ignore when tracking usage value changes                                                                                                                     |
| `report_raw_ids`             |                       `[]`                        | List of IDs to force reporting raw values for. Append `!` to an ID to consider changes critical.                                                                                  |
| `ata_pending_id`             |                      `197+`                       | ID specifying pending sectors. Append `+` to only report increasage instead of non-zero-ness.                                                                                     |
| `ata_uncorrectable_id`       |                      `198+`                       | ID specifying pending sectors. Append `+` to only report increasage instead of non-zero-ness.                                                                                     |
| `temperature_report_diff`    |                       `10`                        | Difference in temperature to report. Use `0` to ignore.                                                                                                                           |
| `temperature_report_info`    |                       `45`                        | Temperature threshold to start sending informational reports. Use `0` to ignore.                                                                                                  |
| `temperature_report_crit`    |                       `50`                        | Temperature threshold to start sending critical reports. Use `0` to ignore.                                                                                                       |
| `firmware_bugs`              |                       `[]`                        | List of known firmware bugs SMARTD should circumvent.                                                                                                                             |
| `vendor_formats`             |                       `[]`                        | List of options to rewrite interpretation of raw SMART values and their interpretation.                                                                                           |
| `preset_mode`                |                       `use`                       | Set to `ignore` if you don't want to use known presets for a detected drive.                                                                                                      |

## Example Playbook

```yml
- hosts: storage
  roles:
    - role: smartd
      smartd_devices:
        /dev/sda:
          check_type: permissive
          test_schedule: L/../../7/04
          temperature_report_diff: 5
        DEVICESCAN:
          non_smart_settings:
           - lookahead,on
           - wcache,off
           - standby,off
           - apm,254
```

This configuration will force SMART checking even if it is not advertised as implemented on `/dev/sda`. The device will only be tested after 4am every sunday and already temperature changes of 5 degrees will be reported.
All other drives will be monitored using the default settings, except for the listed options set additionally.

## License

This work is licensed under a [Creative Commons Attribution-ShareAlike 4.0 International License](https://creativecommons.org/licenses/by-sa/4.0/).


## Author Information

- [Michel Weitbrecht (SlothOfAnarchy)](https://github.com/SlothOfAnarchy) _michel.weitbrecht@stuvus.uni-stuttgart.de_

## Contributors
- [Milos Kaurin (Kaurin)](https://github.com/Kaurin)
