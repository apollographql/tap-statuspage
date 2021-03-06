# tap-statuspage
[![PyPI version](https://badge.fury.io/py/tap-pagerduty.svg)](https://badge.fury.io/py/statuspage)
[![Build Status](https://travis-ci.com/goodeggs/tap-pagerduty.svg?branch=goodeggs/prod)](https://travis-ci.com/goodeggs/tap-statuspage)
[![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue.svg)](https://www.gnu.org/licenses/gpl-3.0)

A [Singer](https://www.singer.io/) tap for extracting data from the [Statuspage REST API v1](https://developer.statuspage.io/).

## Installation

Since package dependencies tend to conflict between various taps and targets, Singer [recommends](https://github.com/singer-io/getting-started/blob/master/docs/RUNNING_AND_DEVELOPING.md#running-singer-with-python) installing taps and targets into their own isolated virtual environments:

### Install Statuspage Tap

```bash
$ make prod-env
```

## Configuration

The tap accepts a JSON-formatted configuration file as arguments. This configuration file has three required fields:

1. `token`: A valid [Statuspage REST API key](https://developer.statuspage.io/#section/Authentication).
2. `start` A date to be used as the default `start` parameter for all API endpoints that support that parameter.

An bare-bones Statuspage configuration may file may look like the following:

```json
{
  "start": "2019-01-01T00:00:00Z",
  "api_key": <api_key>
}
```

## Streams

The current version of the tap syncs three distinct [Streams](https://github.com/singer-io/getting-started/blob/master/docs/SYNC_MODE.md#streams):
1. `Pages`: ([Endpoint](https://developer.statuspage.io/#operation/getPages), [Schema](tap_statuspage/schemas/pages.json))
2. `Incidents`: ([Endpoint](https://developer.statuspage.io/#operation/getPagesPageIdIncidents), [Schema](tap_statuspage/schemas/incidents.json))
3. `Components`: ([Endpoint](https://developer.statuspage.io/#operation/getPagesPageIdComponents), [Schema](tap_statuspage/schemas/components.json))
4. `Uptime`: ([Endpoint](https://developer.statuspage.io/#operation/getPagesPageIdComponentsComponentIdUptime), [Schema](tap_statuspage/schemas/uptime.json))

## Discovery

Singer taps describe the data that a stream supports via a [Discovery](https://github.com/singer-io/getting-started/blob/master/docs/DISCOVERY_MODE.md#discovery-mode) process. You can run the Statuspage tap in Discovery mode by passing the `--discover` flag at runtime:

```bash
$ ./.venvs/tap-statuspage/bin/tap-statuspage --config=config/config.json --discover
```

The tap will generate a [Catalog](https://github.com/singer-io/getting-started/blob/master/docs/DISCOVERY_MODE.md#the-catalog) to stdout. To pass the Catalog to a file instead, simply redirect it to a file:

```bash
$ ./.venvs/tap-statuspage/bin/tap-statuspage --config=config/config.json --discover > catalog.json
```

## Sync Locally

Running a tap in [Sync mode](https://github.com/singer-io/getting-started/blob/master/docs/SYNC_MODE.md#sync-mode) will extract data from the various Streams. In order to run a tap in Sync mode, pass a configuration file and catalog file:

```bash
$ ./.venvs/tap-statuspage/bin/tap-statuspage --config=config/config.json --catalog=catalog.json
```

The tap will emit occasional [State messages](https://github.com/singer-io/getting-started/blob/master/docs/SPEC.md#state-message). You can persist State between runs by redirecting State to a file:

```bash
$ ./.venvs/tap-statuspage/bin/tap-statuspage --config=config/config.json --catalog=catalog.json >> state.json
$ tail -1 state.json > state.json.tmp
$ mv state.json.tmp state.json
```

To pick up from where the tap left off on subsequent runs, simply supply the [State file](https://github.com/singer-io/getting-started/blob/master/docs/CONFIG_AND_STATE.md#state-file) at runtime:

```bash
$ ./.venvs/tap-statuspage/bin/tap-statuspage --config=config/config.json --catalog=catalog.json --state=state.json >> state.json
$ tail -1 state.json > state.json.tmp
$ mv state.json.tmp state.json
```

## Sync to Stitch

You can also send the output of the tap to [Stitch Data](https://www.stitchdata.com/) for loading into the data warehouse. To do this, first create a JSON-formatted configuration for Stitch. This configuration file has two required fields:
1. `client_id`: The ID associated with the Stitch Data account you'll be sending data to.
2. `token` The token associated with the specific [Import API integration](https://www.stitchdata.com/docs/integrations/import-api/) within the Stitch Data account.

An example configuration file will look as follows:

```json
{
  "client_id": 1234,
  "token": "foobarfoobar"
}
```

Once the configuration file is created, simply pipe the output of the tap to the Stitch Data target and supply the target with the newly created configuration file:

```bash
$ ./.venvs/tap-statuspage/bin/tap-statuspage --config=config/config.json --catalog=catalog.json --state=state.json | ./.venvs/target-stitch/bin/target-stitch --config=config/stitch.config.json >> state.json
$ tail -1 state.json > state.json.tmp
$ mv state.json.tmp state.json
```

## Contributing

### Required Tools [Pipenv] (https://docs.pipenv.org/en/latest/) and [Direnv](https://direnv.net/)

The first step to contributing is getting a copy of the source code, and setting up your environment. First, [fork `tap-statuspage` on GitHub](https://github.com/apollographql/tap-statuspage/fork). Then, `cd` into the directory where you want your copy of the source code to live and clone the source code enabling direnv:

```bash
$ git clone git@github.com:YourGitHubName/tap-statuspage.git
$ direnv allow
```

For example, to format your code using [isort](https://github.com/timothycrosley/isort) and [flake8](http://flake8.pycqa.org/en/latest/index.html) before commiting changes, run the following commands:

```bash
$ make isort
$ make flake8
```

You can also run the entire testing suite before committing using [tox](https://tox.readthedocs.io/en/latest/):

```bash
$ make lint
```

Finally, you can run your local version of the tap within the virtual environment using a command like the following:

```bash
$ tap-statuspage --config=config/config.json --catalog=catalog.json
```

There's also a few useful shortcuts avainable in the `Makefile` take a look!

Once you've confirmed that your changes work and the testing suite passes, feel free to put out a PR!
