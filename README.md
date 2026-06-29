# Aidnem Nushell Init Loader

The Aidnem Nushell Loader is a caching script to help speed up startup times on slower machines.

Many programs, (for example, starship, zoxide, and carapace) are initialized by generating an init file and then loading that file. To speed up startup times, these init files can be cached. The cost of this is that, for programs like carapace that depend on regenerating the cache with the latest data, it will need to be regenerated whenever the configuration changes (e.g. downloading new software or getting new completions). To help with this, use `aidnem_loader_remove_file [names...]`

I wrote this program because, after being astonished at Nushell's 30ms load time (after 1500ms for Powershell), it quickly slowed down again after installing starship, zoxide, and carapace. Plus, I wanted to learn nushell scripting.

## Installing

Download `loader.nu` and place it somewhere (I recommend in `~/.config/nushell`). Then, at the end of your `config.nu`, load the file:

```nu
# Run the aidnem loader, last so that things like carapace are pre-configured
source ~/.config/nushell/loader.nu
```

Next, move all of your setup commands that generate files from `env.nu` and `config.nu` to `loader.nu` inside of the configs list.

For example, these configs for starship and zoxide:

```nu
# env.nu
zoxide init nushell | save -f ~/.zoxide.nu

# config.nu
source ~/.zoxide.nu

mkdir ($nu.data-dir | path join "vendor/autoload")
starship init nu | save -f ($nu.data-dir | path join "vendor/autoload/starship.nu")
```

Became:

```nu
let aidnem_loader_configs = [
  {name: 'starship', command: "starship init nu" }
  {name: 'zoxide', command: "zoxide init nushell" }
  {name: 'carapace', command: "carapace _carapace nushell"}
]
```

Note that this will result in files being saved to `$nu.data-dir/vendor/autoload/{name}.nu` rather than the original locations based on their install instructions. This might cause problems for some software, so bear it in mind when debugging.

## Results

How much did it speed up my startup times?

Here's a benchmark where all files are deleted, then the startup is timed while generating the files:

```
➜ 1..10 | each {|_|
  aidnem_loader_remove_file starship zoxide carapace
  timeit { nu --config $nu.config-path --env-config $nu.env-path -c "exit" }
} | math avg
... (Lots of spammed output of deleting and re-adding files)
397ms 214µs 880ns
```

Now here's the startup time with the cache:

```
➜ 1..10 | each { timeit { nu --config $nu.config-path --env-config $nu.env-path -c "exit" } } | math avg
61ms 286µs 40ns
```

TLDR; 397ms -> 61ms. **84.6% reduction in startup time**.
