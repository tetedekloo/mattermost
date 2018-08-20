Mattermost on Flynn
===================

This is a small project to deploy [Mattermost](https://mattermost.com/) to
a [Flynn](http://flynn.io/) cluster.

## Setup

Create an application as usual and set the available disk space to 200MB:

```bash
flynn create mattermost
flynn limit set web temp_disk=200MB
```

And add a PostgreSQL appliance: 

```bash
flynn resource add postgres
```

## Initial Deployment and Updates

Push this repository towards Flynn:

```bash
git push flynn master
```

## Version and Type

By default, version 5.2.0 is installed as type "team". You can override this by setting the environment variables `VERSION` (any valid Mattermost version) and `TYPE` ("team" or "enterprise").

## Updates

Just set the environment variable `TYPE` to the version to update to. A redeployment will happen.

## Further Configuration

*Please note that the configuration set in the System Console is not persistent and won't survive a restart of the Flynn container! All configuration must be done via environment variables!*

Any setting from the configuration json file can be overwritten via environment variables. They have a specific scheme to address the configuration key:

`MM_<rootKey>_<childKey>(_<moreChildKeys)`

So for example to override the `SiteURL` from the `ServiceSettings`, this environment variable would be used: `MM_ServiceSettings_SiteURL`.

There is one gotcha with the values of the environment variables. They have to be valid JSON values. So a string must be wrapped into double quotes for example.
So this would set the `SiteURL` to "https://foo.bar":

```bash
flynn env set MM_ServiceSettings_SiteURL="\"https://foo.bar\""
```

All rules applying JSON values are to be taken into account here. So plain numbers can stay the way they are.
If this is done wrong, the application won't start. If done wrong, just scale back via `flynn scale web=0`, set
the correct value and scale up again.

The ListenAddress and initial database connection have default values generated by the Flynn environment variables:

```bash
export MM_ServiceSettings_ListenAddress="\":$PORT\""
export MM_SqlSettings_DriverName=${MM_SqlSettings_DriverName:-"\"postgres\""}
export MM_SqlSettings_DataSource=${MM_SqlSettings_DataSource:-"\""$DATABASE_URL?sslmode=disable&connect_timeout=10"\""}
```

So they are always present. But the SqlSettings can be overridden just by setting `MM_SqlSettings_DriverName` and `MM_SqlSettings_DataSource`.

See `flynn log` for any other errors if things don't work out as expected. Example: MM_TeamSettings_SiteName can only have a length of 30 characters.

## License

This software is licensed under the
[MIT License](https://opensource.org/licenses/MIT). Adminer itself (index.php)
is licensed under the
[Apache License](https://www.apache.org/licenses/LICENSE-2.0.html).
