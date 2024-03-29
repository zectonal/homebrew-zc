![](images/ZectonalLogo.jpg)

# zc

A blazingly fast Data Observability diagnostic tool that runs in your environment.

View our [License](license/index.html)

&nbsp;

# Table of Contents

- [Capabilities](#capability-summary)
- [Installation](#installation)
- [Configuration](#quick-start)
- [Run](#run)
- [Screenshots](#screenshots)
- [Support and New Feature Requests](#support)

&nbsp;

# Capability Summary

**zc** is a command line diagnostic tool that performs a one-time analysis of a data source and outputs specific data observability attributes.

**zc** is the blazingly fast command line tool written in Rust that is a companion to the flagship [Zectonal](https://www.zectonal.com) **Data Observability and Deep Data Inspection Platform**.

**zc** is perfect for providing a quick snapshot for information about your data sources.

**zc** is also perfect for iterating to generating an optimal [Configuration](#configuration) for the [Zectonal](https://www.zectonal.com) platform.

**zc** only runs in your environment. We believe that your data in someone else's SaaS platform or Cloud is for their benefit and not yours.

&nbsp;

# Installation

&nbsp;

#### macOS (brew)

```sh
brew install zc
```

&nbsp;

#### macOS (manual)

1. Download [**zc_0.14.0_macos.zip**](http://zectonal-releases.s3-website-us-east-1.amazonaws.com/zc/releases/download/v0.14.0/zc_0.14.0_macos.zip)

```bash
curl -LO http://zectonal-releases.s3-website-us-east-1.amazonaws.com/zc/releases/download/v0.14.0/zc_0.14.0_macos.zip
```

2. Validate checksum `sha256sum zc_0.14.0_macos.zip`
3. Unzip `unzip zc_0.14.0_macos.zip`
4. Run `./zc --configuration-file <your TOML config file>`

&nbsp;

# Configuration

**zc** uses a TOML-formatted file for configuring data sources

A **Configuration** is comprised of two items:

1. [Environments](#environments)
1. [Sources](#sources)

&nbsp;

### Environments

Environments contain information about the broader context of the data source and are intended to be re-usable with one or more [Sources](#sources).

&nbsp;

**Posix Environment**

```toml
[environments]

[environments.my_posix_env]
env_type = "posix"
```

**note**: the `env_type` must be `posix`

**note**: the `name` in this case is `my_posix_env` and must match exactly the `env_name` variable found in the [Sources](#sources) section (see below)

&nbsp;

**AWS Environment**

```toml
[environments.my_aws_env]
env_type = "aws"
credential_profile_name = "not-default"
```

**note**: the `env_type` must be `aws`

**note**: the `name` in this case is `my_aws_env` and must match exactly the `env_name` variable found in the [Sources](#sources) section (see below)

&nbsp;

### Sources

#### Minimal Configuration

&nbsp;

**Posix minimum configuration**

```toml
[sources]

[sources.Downloads]
env_name = "my_posix_env"
base_location = "fs:///Users/me/Downloads"
poll_frequency = "1m"
```

**note**: the default name for this Source is `Downloads`

**note**: the `base_location` follows industry standard URI schemes and so for a Posix Source it must start with `fs://`

**note**: the `env_name` must match exactly the `name` found in the [Environments](#environments) section ([see above](#environments))

&nbsp;

**AWS minimum configuration**

```toml
[sources.zect_test]
env_name = "my_aws_env"
base_location = "s3://my-test-bucket"
aws_region = "us-east-1"
poll_frequency = "2m"
```

Further, the AWS confiuration supports an optional parameter called `sub_directory`

```toml
sub_directory = "output/"
```

**note**: the default name for this Source is `zect_test`

**note**: the `base_location` for an AWS S3 Source must start with `s3://` URI scheme.

**note**: the `env_name` must match exactly the `name` found in the [Environments](#environments) section ([see above](#environments))

&nbsp;

#### Extended Configuration

The following _optional_ configuration parameters can be used for both Posix and AWS S3 Sources. Append to the Sources section as required.

The `alias` parameter overrides the name of the Source.

```toml
alias = "Downloads"
```

&nbsp;

**Last Observed Threshold Analysis**

Last Observed Threshold Analysis is a feature that allows you to set a threshold for the last observed time of a file. If the last observed time of a file is older than the threshold, then an `Event` will be created.

This is helpful when you want to be alerted when data is `stale` or `late`.

```toml
enable_last_observed_threshold = true
last_observed_threshold = "2h"
```

&nbsp;

**File Query Analysis**

File Query Analysis is a feature that allows you to query the content of files directly, including **parquet**, **csv**, **csv.gz**, **tsv**, **tsv.gz**, and **avro** files.

By setting `enable_file_query` to `true`, you can query the content of files using the `file_query_lookback_period` to specify the time period to query. In the example below, the application will query the content of all files that are less than 6 days old.

```toml
enable_file_query = true
file_query_lookback_period = "6d"
```

&nbsp;

**File Count Analysis**

File Count Analysis is a feature that allows you to set a threshold for the number of files in a directory. **zc** If the number of files in a directory is outside the threshold, then an `Event` will be created.

File Count Analysis can be configured to be `static` or `dynamic`. Static file count analysis uses fixed minimum and maximum thresholds, while dynamic file count analysis uses a sigma threshold based on the mean and standard deviation of the number of files in the directory.

```toml
enable_file_count_check = true
enable_dynamic_file_count = true
dynamic_file_count_sigma_threshold = 2
enable_static_file_count = true
min_num_file_count_threshold = 1
max_num_file_count_threshold = 3
file_count_lookback_period = "10d"
```

&nbsp;

**Text Search Analysis**

Text Search Analysis is a feature that allows you to search for specific strings in the content of files. If the string is found, then an `Event` will be created.

Text Search Analysis requires that `enable_file_query` is set to `true` and that the `search_strings` array is populated with the strings to search for.

```toml
enable_text_search_detection = true
search_strings = ["ldap://log4jshellserver.com", "needle", "haystack"]
```

&nbsp;

# Run

A growing number of command line options are available to customize the behavior of `zc`. They can be used independently or in combination.

&nbsp;

##### configuration file

In the absence of a configuration file, either explicitly via `--configuration-file <FILE>` flag or implicitly via the default location `$XDG_HOME/.config/zect/config.toml`, `zc` will default to using both `$HOME/Downloads` and `$HOME/Documents`.

```sh
zc --configuration-file <path to TOML config file>
```

&nbsp;

##### query files

To query files, use the `--q` or the `--query-files` flag. Querying files can take a long time depending on the size of the files and the number of files, so use this feature judiciously.

```bash
zc --q
zc --query-files
```

&nbsp;

##### show active events

To only show active events, use the `--active` flag

```bash
zc --show-active-events
```

&nbsp;

##### hide logo

To hide the text-based splash screen, use the `--hide-logo` flag

```sh
zc --hide-logo
```

&nbsp;

##### help

To display all command line options, use the `--help` flag

```sh
zc --help
```

&nbsp;

# Screenshots

![](images/zc.gif)

&nbsp;

# Support

Email us at [support@zectonal.com](mailto:support@zectonal.com)

Join a discussion on our `Zectonal` [Discord](https://www.discord.com) Server
