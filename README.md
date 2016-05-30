config package for R
================

The **config** package makes it easy to manage environment specific configuration values For example, you might want to use distinct values for development, testing, and production environments.

Usage
-----

Configurations are defined using a [YAML](http://www.yaml.org/about.html) text file and are stored (by default) in a file named **config.yml**. Configuration files include default values as well as values for arbitrary other named configurations, for example:

**config.yml**

``` yaml
default:
  trials: 5
  dataset: "data-sampled.csv"
  
production:
  trials: 30
  dataset: "data.csv"
```

To read values from this configuration file you call the `config::get` function, which returns a list containing all of the values for the currently active configuration:

``` r
config <- config::get()
config$trials
config$dataset
```

You can also read a single value from the configuration as follows:

``` r
config::get("trials")
config::get("dataset")
```

The `get` function takes a `config` argument which determines which configuration to read values from. By default, `config` is read from the `R_CONFIG_NAME` environment variable. This variable is in turn typically set within a site-wide `Renviron` or `Rprofile` (see [R Startup](https://stat.ethz.ch/R-manual/R-devel/library/base/html/Startup.html) for details on these files):

``` r
# set the active configuration globally via Renviron.site or Rprofile.site
Sys.setenv(R_CONFIG_NAME = "production")

# read configuration value (will return 30 from the "production" config)
config::get("trials")
```

Defaults and Inheritance
------------------------

The `default` configuration must define all available values within the configuration file. Other configurations (e.g. `test` or `production`) automatically inherit all `default` values so need only define values specialized for that configuration. For example, in this configuration the `trials` value will be `5` when read from the `production` configuration:

**config.yml**

``` yaml
default:
  trials: 5
  dataset: "data-sampled.csv"
  
production:
  dataset: "data.csv"
```

Configurations can alternatively inherit from other configurations. For example, in this file the `production` configuration inherits from the `test` configuration:

**config.yml**

``` yaml
default:
  trials: 5
  dataset: "data-sampled.csv"

test:
  trials: 30
  dataset: "data-test.csv"
  
production:
  inherits: test
  dataset: "data.csv"
```

Configuration Files
-------------------

By default configuration data is read from a file named **config.yml** within the current working directory. You can use the `file` argument of `config::get` to read from an alternate location. For example:

``` r
config <- config::get(file = "conf/config.yml")
```

Sometimes it's useful to define local configuration values that are not deployed to other systems or not checked into version control. The **config** package will look for a file with a `.local.` extension prefix (e.g. **config.local.yml**) and use it's contents as an override of what's found in the main configuration file.

Dynamic Values
--------------

-   Executing R code