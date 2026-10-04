# TOFIX

Findings from a code scan on 2026-10-04.

## Medium

- `install_awscli.sh:14` - `pip uninstall awscli` has no `-y`, so it stops at the "Proceed (Y/n)?" prompt (or fails with no tty), and `2> /dev/null || true` hides the failure while the script then reports success; use `pip uninstall -y awscli` and drop `|| true`.
- `install_packages.sh:2` - installs only the python packages, but the other scripts need `curl` and `unzip` (`install_awscli.sh:5-6`) and `dig` (`myip.sh:2`, package `bind9-dnsutils`), which are absent on minimal cloud images; add them.
- `config_homedir.sh:3` - `cat sourceme >> ~/.bashrc` reads `sourceme` relative to the current directory (fails unless run from the repo root) and appends a duplicate block on every run; resolve the path from `$(dirname "$0")` and skip if already present.
- `install_awscli.sh:5` - the download URL is hard-coded to `awscli-exe-linux-x86_64.zip`, so the script installs a non-runnable binary on ARM (Graviton) instances; pick `x86_64`/`aarch64` from `uname -m`.

## Low

- `update_everything.sh:3-4` - blanking `/etc/apt/apt.conf.d/20apt-esm-hook.conf` edits a package-owned conffile (dpkg will prompt on the next `ubuntu-advantage-tools`/`ubuntu-pro-client` upgrade); use the supported `sudo pro config set apt_news=false` that is commented out on line 5.
- `update_everything.sh:9` - `apt upgrade -y` after `apt dist-upgrade -y` is a no-op; drop it.
- `update_everything.sh:2` - typos "ridd" and "commericals".
- `sourceme:2` - unconditionally sources `~/.venv/bin/activate`, so every new shell prints an error until `install_virtualenv.sh` has been run; guard it with `[ -f ... ]`.
- `README.md:2` - no description of the scripts or the order to run them in (`install_packages.sh` -> `install_virtualenv.sh` -> `install_awscli.sh` -> `config_homedir.sh`); document it.
