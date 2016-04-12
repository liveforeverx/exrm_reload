# ExrmReload [![Build Status](https://travis-ci.org/xerions/exrm_reload.svg)](https://travis-ci.org/xerions/exrm_reload)

Build new sys.config from confrom config and apply it at runtume.
It uses `conform_schema` and `conform_config` command line flags which are set on [exrm](https://github.com/bitwalker/exrm) startup script.

## Usage

1. Add exrm_reload to your list of dependencies in mix.exs:

    ```elixir
    def deps do
      [{:exrm_reload, github: "xerions/exrm_reload"}]
    end
    ```

2. Ensure exrm_reload is started before your application:

    ```elixir
    def application do
      [applications: [:exrm_reload]]
    end
    ```

3. Run reconfiguration when you want it:

	```elixir
	> ReleaseManager.Reload.run
	```

	or you can specify application's list for reconfiguration:

	```elixir
	> ReleaseManager.Reload.run [:hello, :exd, :ecdo]
	```

It works with the releases are builded via `exrm`. You just call it by rpc from OS shell:

	$ you_application rpc ReleaseManager.Reload run

The test application uses stable versions of `exrm` (`1.0`), `conform` (`2.0`), `conform_exrm` (`1.0`). It is not garantied to work with lower versions.

## Usage for developing

1. It is possible to use for developing:

    ```elixir
    config :exrm_reload,
      watch: false,
      dev: Mix.env == :dev
    ```

2. Add this 5-liner to your `config/dev.exs` (it can be used only for `dev` enviroment) to use conform configuration reloaded in development mode.

    ```elixir
    app = Mix.Project.get!.project[:app]
    Mix.Project.build_path |> Path.join("/lib/conform/ebin") |> to_char_list() |> :code.add_path()
    {:ok, conf} = Conform.Conf.from_file(config)
    config = "config/#{app}.schema.exs" |> Conform.Schema.load! |> Conform.Translate.to_config([], conf)
    for {app, envs} <- config, do: config(app, envs)

    config :exrm_reload,
      watch: true,
      dev: true
    ```
