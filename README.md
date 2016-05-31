config package for R
================
J.J. Allaire
2016-05-31

The **config** package makes it easy to manage environment specific configuration values. For example, you might want to use distinct values for development, testing, and production environments.

Usage
-----

Configurations are defined using a [YAML](http://www.yaml.org/about.html) text file and are read by default from a file named **config.yml** in the current working directory (or parent directories if no config file is found in the initially specified directory).

Configuration files include default values as well as values for arbitrary other named configurations, for example:

**config.yml**

``` yaml
default:
  trials: 5
  dataset: "data-sampled.csv"
  
production:
  trials: 30
  dataset: "data.csv"
```

To read configuration values you call the `config::get` function, which returns a list containing all of the values for the currently active configuration:

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

The `get` function takes an optional `config` argument which determines which configuration to read values from (the "default" configuration is used if none is specified).

The active configuration should typically specified globally via the `R_CONFIG_NAME` environment variable rather than within the call to `get`:

``` r
# set the active configuration globally via Renviron.site or Rprofile.site
Sys.setenv(R_CONFIG_NAME = "production")

# read configuration value (will return 30 from the "production" config)
config::get("trials")
```

Note that this variable is typically set within a site-wide `Renviron` or `Rprofile` (see [R Startup](https://stat.ethz.ch/R-manual/R-devel/library/base/html/Startup.html) for details on these files).

Defaults and Inheritance
------------------------

The `default` configuration must define all values available within the configuration file. Other configurations automatically inherit all `default` values so need only define values specialized for that configuration. For example, in this configuration the `production` configuration doesn't specify a value for `trials` so it will be read from the `default` configuration:

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

By default configuration data is read from a file named **config.yml** within the current working directory (or parent directories if no config file is found in the initially specified directory).

You can use the `file` argument of `config::get` to read from an alternate location. For example:

``` r
config <- config::get(file = "conf/config.yml")
```

If you don't want to ever scan parent directories for configuration files then you can specify `use_parent = FALSE`:

``` r
config <- config::get(file = "conf/config.yml", use_parent = FALSE)
```

### Local Configs

Sometimes it's useful to define local configuration values that are not deployed to other systems or not checked into version control. The **config** package will look for a file with the `.local.` extension prefix (e.g. **config.local.yml**) and use values defined within it to override values of the same name found in the main configuration file.

R Code
------

You can execute R code within configuration files by prefacing values with `!expr`. This could be useful in the case where you want to base configuration values on environment variables, R options, or even other config files. For example:

``` yaml
default:
  cores: 2
  debug: true
  server: "localhost:5555"
   
production:
  cores: !expr getOption("mc.cores")
  debug: !expr Sys.getenv("ENABLE_DEBUG") == "1"
  server: !expr config::get("server", file = "/etc/server-config.yml")
```