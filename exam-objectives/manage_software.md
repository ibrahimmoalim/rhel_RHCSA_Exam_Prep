# Manage software ✅

## Configure access to RPM repositories ✅
### Clear out existing RPM repos if any (to practice configuring them)
- RHEL reads repository configurations from `.repo` files located inside the `/etc/yum.repos.d/` directory. To reset your system:
    - do the following:
    First disable subscription manager plugin so it doesn't automatically regenerate `redhat.repo` file which allows you to install packages.
    ```bash
    sudo vi /etc/dnf/plugins/subscription-manager.conf
    ```
    Look for the line `enabled=1` and change it to `0`:
    ```bash
    [main]
    enabled=0
    ```
    save and exit (:wq).
    Now clear any repos:
    ```bash
    cd /etc/yum.repos.d/
    ```
    Check what files are in there (they end in `.repo`), then either move them to a backup place or delete them:
    ```bash
    sudo rm -rf /etc/yum.repos.d/*
    ```
    Make sure the dir is empty, then clear cached repos:
    ```bash
    sudo dnf clean all
    ```
    Now check `repolist`
    ```bash
    sudo dnf repolist
    ```
    > it should be empty, and you shouldn't be able to update the system or install any packages
### Configure access to RPM repositories
- create a `.repo` file in `/etc/yum.repos.d/`
```bash
# you can give it anyname but 'local' best defines it
sudo vi /etc/yum.repos.d/local.repo
```
- put this into that file
```Ini, TOML
# unless you're told to directly give it specific name
# you can give it any name
[BaseOS]
name=BaseOS
baseurl=<baseOS url provided in the exam>
enabled=1
gpgcheck=0

[AppStream]
name=AppStream
baseurl=<appStream url provided in the exam>
enabled=1
gpgcheck=0
```
save and quit
- clear old data if any
```bash
sudo dnf clean all
```
- check if the repo list to see those newly added repos
```bash
sudo dnf repolist
```

## Install and remove RPM software packages ✅
- `sudo dnf update -y` => update system
- `sudo dnf install <pkg-name> -y` => install a pkg
- `sudo dnf remove <pkg-name> -y` => removes a pkg

## Configure access to Flatpak repositories
- first install flatpak if not installed
```bash
sudo dnf install flatpak -y
```
- add flatpak repo (if not configured alraedy and exam tells you to configure)
```bash
# e.g flatpak remote-add --if-not-exists flathub http://...
flatpak remote-add --if-not-exists <any-name> <url-given>
```
- verify
```bash
flatpak remotes
```
> You should see the name you chose
- update flatpak to sync metadata (important step)
```bash
flatpak update -y
```

## Install and remove Flatpak software packages ✅
- `flatpak search <app-name> => search for an app.
- for the below run,install,uninstall it's best to use full app-ID
- `flatpak install <app-name>` => installs app, you'll be given options to choose the correct one, you can also use the `application ID` seen in search results.
- `flatpak run <app-ID>` => run/verify the flatpak app, e.g `flatpak run com.discordapp.Discord` (you have to use ID, not name).
- `flatpak uninstall <app-name>` => uninstall app, you can use `app-ID` too.