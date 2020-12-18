# Nested changes

**Please read the documentation below to get an understanding of the functions of the components in this repo. Instruction here are just to run the components**

This repo holds the components that load data into the BQ dataset.

The main project we need is the `loader`, this is a streaming DataFlow job to load the enriched data from the Pub/Sub into BQ.

`mutator` and `repeater` might be needed if events have special context/schemas. `mutator`, changes the schema of the BQ table to allow for said events to be written, and `repeater` retries inserts after the table has the right/expected format.

Both `mutator` and `repeater` do not run on DataFlow and would have to be run ad-hoc on local machine every so often or provision a compute instance to run them, or lastly maybe port them to DataFlow.

## Pre-Requisites

- Have `sbt` installed
- Have JRE (Java Runtime Engine) on your machine
- Have access to the GCP console

_Note: Access to the GCP console should be a given if you work on nested projects, hence not covered here_

### Install sbt

This can be achieve with `asdf`.

```sh
# First install the plugin
asdf plugin add sbt https://github.com/bram2000/asdf-sbt.git

# Then install the sbt version 1.3.10
asdf install sbt 1.3.10
```

### JRE

You should have JRE installed on the machine, if this is not the case but you have installed `sbt` with the above method you should see a link to download JRE.

If not just go onto http://java.com/en/download

## Build

To build projects use the sbt command line, and the following example

```sh
sbt "project loader" universal"packageBuild
```

This will create a `target/universal` folder inside the `loader` project, which should have a zip file. This is the project packaged with helpful scripts to run it.

## Run

To run the projects we need to unpack that zip file that we built and use the script inside the `bin` folder that goes by the same name as the project.

Here are the commands to run each project

### loader

```sh
./bin/snowplow-bigquery-loader \
--runner=DataFlowRunner \
--project=steadfast-range-119414 \
--streaming=true \
--region=europe-west1 \
--gcpTempLocation=gs://nested-snowplow-test/tmp/ \
--job-name=<whatever_name_you_like> \
--resolver=$(cat iglu_resolver.json | base64) \
--config=$(cat test_config.json | base64)
```

This should all be self-explanatory, and yes 2 of the params need to be base64 encoded, because 🤷

### mutator

```sh
./bin/snowplow-bigquery-mutator \
create \
--resolver=$(cat iglu_resolver.json | base64) \
--config=$(cat test_config.json | base64)
```

### repeater

```sh
./bin/snowplow-bigquery-repeater \
--resolver $(cat iglu_resolver.json | base64) \
--config $(cat test_config.json | base64) \
--failedInsertsSub snowplow-repeater \
--deadEndBucket gs://nested-snowplow-test/loader-dead-end/ \
--desperatesBufferSize 2 \
--desperatesWindow 20 \
--backoffPeriod 60
```

# Snowplow BigQuery Loader

[![Build Status][travis-image]][travis]
[![Release][release-image]][releases]
[![License][license-image]][license]

This project contains applications used to load [Snowplow][snowplow] enriched data into [Google BigQuery][bigquery].

## Quickstart

Assuming git and [SBT][sbt] installed:

```bash
$ git clone https://github.com/snowplow-incubator/snowplow-bigquery-loader
$ cd snowplow-bigquery-loader
$ sbt "project loader" test
$ sbt "project streamloader" test
$ sbt "project mutator" test
$ sbt "project repeater" test
```

## Find out more

| **[Technical Docs][techdocs]**    | **[Setup Guide][setup]**    | **[Contributing][contributing]**          |
| --------------------------------- | --------------------------- | ----------------------------------------- |
| [![i1][techdocs-image]][techdocs] | [![i2][setup-image]][setup] | [![i3][contributing-image]][contributing] |

## Copyright and License

Snowplow BigQuery Loader is copyright 2018-2020 Snowplow Analytics Ltd.

Licensed under the **[Apache License, Version 2.0][license]** (the "License");
you may not use this software except in compliance with the License.

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.

[snowplow]: https://github.com/snowplow/snowplow/
[bigquery]: https://cloud.google.com/bigquery/
[sbt]: https://www.scala-sbt.org/
[license-image]: http://img.shields.io/badge/license-Apache--2-blue.svg?style=flat
[license]: http://www.apache.org/licenses/LICENSE-2.0
[travis]: https://travis-ci.org/snowplow-incubator/snowplow-bigquery-loader
[travis-image]: https://travis-ci.org/snowplow-incubator/snowplow-bigquery-loader.png?branch=master
[release-image]: http://img.shields.io/badge/release-0.6.1-blue.svg?style=flat
[releases]: https://github.com/snowplow-incubator/snowplow-bigquery-loader
[techdocs-image]: https://d3i6fms1cm1j0i.cloudfront.net/github/images/techdocs.png
[setup-image]: https://d3i6fms1cm1j0i.cloudfront.net/github/images/setup.png
[contributing-image]: https://d3i6fms1cm1j0i.cloudfront.net/github/images/contributing.png
[techdocs]: https://docs.snowplowanalytics.com/docs/setup-snowplow-on-gcp/setup-bigquery-destination/bigquery-loader-0-6-0/
[setup]: https://docs.snowplowanalytics.com/docs/setup-snowplow-on-gcp/setup-bigquery-destination/bigquery-loader-0-6-0/#setup-guide
[contributing]: https://github.com/snowplow/snowplow/wiki/Contributing
