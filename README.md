# What is **zc**?

A blazingly fast Data Observability command line diagnostic tool written in Rust that runs in your environment.

![](images/ZectonalLogo.jpg)

# Why **zc**

- blazingly fast and a companion to the [Zectonal Data Observability and Deep Data Inspection Platform<sup>(c)</sup>](https://www.zectonal.com)

- provides a quick snapshot for observability information about your data sources

- perfect for iterating to generating an optimal [Configuration](#configuration) for the [Zectonal](https://www.zectonal.com) platform

- only runs in your environment

Data observability metrics in someone else's SaaS platform or Cloud is for their benefit and not yours

# Connect with Us

Zectheads Unite!

Who Are You? Where Are You?

Email us at [support@zectonal.com](mailto:support@zectonal.com)

Join a discussion on our [Zectonal Discord Server](https://www.discord.com)

View our [License](license.md)

# Table of Contents

- [Installation](#installation)
- [Configuration](#quick-start)
- [Quickstart](#run)
- [Screenshots](#screenshots)

# Installation

#### macos (brew) install (recommended)

```bash
brew tap zectonal/zc
brew update
brew install zc
```

#### macos (brew) uninstall (recommended)

```bash
brew remove zc
brew untap zectonal/zc
```

#### macOS (manual) installation

1. Download [**zc_0.14.0_macos.zip**](http://zectonal-releases.s3-website-us-east-1.amazonaws.com/zc/releases/download/v0.14.0/zc_0.14.0_macos.zip)

```bash
curl -LO http://zectonal-releases.s3-website-us-east-1.amazonaws.com/zc/releases/download/v0.14.0/zc_0.14.0_macos.zip
```

2. Validate checksum

```bash
sha256sum zc_0.14.0_macos.zip
```

3. Unzip

```bash
unzip zc_0.14.0_macos.zip
```

4. Run

```bash
./zc --configuration-file <your optional TOML config file>
```

# Configuration

A **Configuration** is comprised of two items:

1. [Environments](#environments)
1. [Sources](#sources)

> [!Tip]
> zc uses a TOML-formatted file for configuring data sources


### Environments

Environments contain information about the broader context of the data source and are intended to be re-usable with one or more [Sources](#sources).

**Posix Environment**

```toml
[environments]

[environments.my_posix_env]
env_type = "posix"
```

> [!Important]
>
> - the `env_type` must be `posix`
> - the `name` in this case is `my_posix_env` and must match exactly the `env_name` variable found in the [Sources](#sources) section (see below)


**AWS Environment**

```toml
[environments.my_aws_env]
env_type = "aws"
credential_profile_name = "not-default"
```

> [!Important]
>
> - the `env_type` must be `aws`
> - the `name` in this case is `my_aws_env` and must match exactly the `env_name` variable found in the [Sources](#sources) section (see below)


### Sources

#### Minimal Configuration

**Posix minimum configuration**

```toml
[sources]

[sources.Downloads]
env_name = "my_posix_env"
base_location = "fs:///Users/me/Downloads"
poll_frequency = "1m"
```

> [!Tip]
> the default name for this Source is `Downloads` based on TOML syntax


> [!Important]
>
> - the `base_location` follows industry standard URI schemes and so for a Posix Source it must start with `fs://`
> - the `env_name` must match exactly the `name` found in the [Environments](#environments) section ([see above](#environments))


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

> [!Tip]
> the default name for this Source is `zect_test` based on TOML syntax


> [!Important]
>
> - the `base_location` for an AWS S3 Source must start with `s3://` URI scheme.
> - the `env_name` must match exactly the `name` found in the [Environments](#environments) section ([see above](#environments))


#### Extended Configuration

The following _optional_ configuration parameters can be used for both Posix and AWS S3 Sources. Append to the Sources section as required.

The `alias` parameter overrides the name of the Source.

```toml
alias = "Downloads"
```

**Last Observed Threshold Analysis**

Last Observed Threshold Analysis is a feature that allows you to set a threshold for the last observed time of a file. If the last observed time of a file is older than the threshold, then an `Event` will be created.

This is helpful when you want to be alerted when data is `stale` or `late`.

```toml
enable_last_observed_threshold = true
last_observed_threshold = "2h"
```

**File Query Analysis**

File Query Analysis is a feature that allows you to query the content of files directly, including **parquet**, **csv**, **csv.gz**, **tsv**, **tsv.gz**, and **avro** files.

By setting `enable_file_query` to `true`, you can query the content of files using the `file_query_lookback_period` to specify the time period to query. In the example below, the application will query the content of all files that are less than 6 days old.

```toml
enable_file_query = true
file_query_lookback_period = "6d"
```

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

**Text Search Analysis**

Text Search Analysis is a feature that allows you to search for specific strings in the content of files. If the string is found, then an `Event` will be created.

Text Search Analysis requires that `enable_file_query` is set to `true` and that the `search_strings` array is populated with the strings to search for.

```toml
enable_text_search_detection = true
search_strings = ["ldap://log4jshellserver.com", "needle", "haystack"]
```

# Quickstart

A growing number of command line options are available to customize the behavior of `zc`. They can be used independently or in combination.

##### configuration file

In the absence of a configuration file, either explicitly via `--configuration-file <FILE>` flag or implicitly via the default location `$XDG_HOME/.config/zect/config.toml`, `zc` will default to using both `$HOME/Downloads` and `$HOME/Documents`.

```bash
zc --configuration-file <path to TOML config file>
```

##### query files

To query files, use the `--q` or the `--query-files` flag. Querying files can take a long time depending on the size of the files and the number of files, so use this feature judiciously.

```bash
zc --q
zc --query-files
```

> [!Caution]
> querying files can take a long time depending on the size and volume of files


##### show active events

To only show active events, use the `--show-active-events` flag

```bash
zc --show-active-events
```

##### hide logo

To hide the text-based splash screen, use the `--hide-logo` flag

```sh
zc --hide-logo
```

##### help

To display all command line options, use the `--help` flag

```sh
zc --help
```

# Screenshots

![](images/zc.gif)
