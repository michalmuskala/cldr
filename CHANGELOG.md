# Changelog for Cldr v1.4.0

This is the changelog for Cldr v1.4.0 released on _, 2018.  For older changelogs please consult the release tag on [GitHub](https://github.com/kipcole9/cldr/tags)

## Enhancements

* The primary focus on this release to automate the process of recompiling those parts of Cldr that are sensitive to configuration change.  This release:

  * Adds a `compiler.cldr` as a Mix compiler.  This provides the functions that identify a locale configuration change and then recompile the relevant modules of Cldr.  Basically it looks in all dependencies for any module that calls functions in `Cldr.Config`  Therefore it is expected to work for Cldr and any package that depends on Cldr.

  * To enable this functionality, the `compiler.cldr` mix task needs to be added in `mix.exs` as the last compiler in the list.  For example

```
  def project do
    [
      app: :app_name,
      ...
      compilers: Mix.compilers ++ [:cldr],
      start_permanent: Mix.env == :prod,
      deps: deps()
    ]
  end
```

  * With this enhancement it will be no longer necessary to force recompile Cldr or packages that depend on it since they will be compiled whenever a locale configuration change is detected.

  * **Please note that the `:cldr` compiler is only available on Elixir version 1.6.0 and later since it depends on `Mix.Tasks.Xref.calls/0`**.  In the case of Elixir versions up to 1.5 a manual `mix deps.compile ex_cldr --force` will be required if the locale configuration changes.

* The [Jason](https://hex.pm/packages/jason) json library is added to the dependency list as an optional dependency.

* Adds `:iso_digits` to the currency data maintained in CLDR.  Cldr currency data does not always align with the ISO definition of a currency - notably for digits (subunit) definitions.  For example, the Colombian Peso has an official subunit of the _pesavo_.  One hundred _pesavos_ equals one _peso_.  There are no notes/cash/coins for the _centavo_ but it is an offical part of the currency and it's important to be maintained for financial transactions.

* Adds a `Mix` task to download the ISO currency data that is used during locale consolidation to include ISO currency digits (subunits) in the currency definition.

## Bug Fixes

* Don't crash if a Gettext backend is defined by it has no configuration for `:default_locale`.  Closes #38.  Thanks to @schultzer.

* Fix incorrect boolean expression, replace with `&&` when determining if `Gettext` is configured

* Create a "null" Cldr compiler so that on Elixir versions less than 1.6 we don't raise an exception if the compiler is configured.

# Changelog for Cldr v1.3.2

This is the changelog for Cldr v1.3.2 released on January 20th, 2018.  For older changelogs please consult the release tag on [GitHub](https://github.com/kipcole9/cldr/tags)

### Bug Fixes

* Correctly retrieve the default locale from a configured Gettext backend module.  Fixes #32.

# Changelog for Cldr v1.3.1

### Bug Fixes

* Correctly use `conn.path_params` not the incorrect `conn.url_params` in `Cldr.Plug.SetLocale` and add plug router tests. Fixes #33.

* Correctly set the default locale using the Gettext default locale if the Cldr default locale isn't set but Gettext is.

# Changelog for Cldr v1.3.0

### Enhancements

* Add `Cldr.Digits.number_of_digits/1` that returns the number of digits (precision) of a float, integer or Decimal.  The primary intent is to support better detection of precision errors after parsing a float string.  A double precision 64-bit float (which is what Erlang/Elixir use) can safely support 15 digits.  According to [Wikipedia](https://en.wikipedia.org/wiki/IEEE_754#Character_representation) a decimal floating point number should round-trip convert to string representation and back for 16 digits without rounding (and 17 using "round to even").  Some examples:

```
iex> Cldr.Digits.number_of_digits(1234)
4

iex> Cldr.Digits.number_of_digits(Decimal.new("123456789"))
9

iex> Cldr.Digits.number_of_digits(1234.456)
7

iex> Cldr.Digits.number_of_digits(1234.56789098765)
15

iex> Cldr.Digits.number_of_digits '12345'
5
```

# Changelog for Cldr v1.2.0

### Bug Fixes

* The changelog refers to the configuration key `json_library` but the readme and the code refer to `json_lib`.  Standardise on `json_library`.  Thanks to @lostkobrakai.

### Enhancements

* Fix the spec for `Cldr.known_currencies/0`.  Thanks to @lostkobrakai.

# Changelog for Cldr v1.1.0

## Enhancements

* When configuring a locale name of the form "language-Script" or "language-Variant" the base "language" is also configured since plural rules will fall back to the base language if the locale name does not contain plural rules.

* When expanding wildcard locale names a more informative exception and error is produced if the regex for the locale names is invalid

* When a locale doesn't have any plural rules the error tuple `{:error, {Cldr.UnknownPluralRules, message}}` is returned instead of the current behaviour which raises an exception.

* The json library is now configurable.  This permits the use of the new json library [jason](https://hex.pm/packages/jason) or any other library that provides `decode!/1` and `encode!/1`.  The library is configured in `config.exs` as follows:

```
config :ex_cldr,
  json_library: Jason # The default is Poison
```

