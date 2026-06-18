## Enabling EPEL for RHEL 10

The EPEL URL is slightly different for version 10, and you must make sure the **CodeReady Linux Builder (CRB)** repository is turned on first, as many EPEL packages depend on it.

Run these to get EPEL 10 up and running:

```bash
# Enable the CRB repo
sudo subscription-manager repos --enable codeready-builder-for-rhel-10-$(arch)-rpms

# Install the EPEL 10 repository configuration
sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-10.noarch.rpm

```

Once that is done, you can install **`htop`** and **`tree`** normally. For system specs, use **`fastfetch` or `neofetch`** if screenfetch is unavailable:

```bash
sudo dnf install htop tree neofetch
```

---

## Ansible on RHEL 10

If you are looking for **`ansible`** on RHEL 10, you have two great paths:

* **The standard CLI engine (`ansible-core`)**: Unlike RHEL 9, `ansible-core` is actually built right into the default RHEL 10 AppStream repositories. You don't have to enable anything extra! Just run:
```bash
sudo dnf install ansible-core
```


* **The Full Ansible Automation Platform**: If you need the massive enterprise suite and extra platform repositories, enable the newer repository string for version 10:
```bash
sudo subscription-manager repos --enable ansible-automation-platform-2.5-for-rhel-10-x86_64-rpms
```



---

## Java (`default-jdk`)

RHEL 10 ships with newer long-term support versions of Java natively. Instead of `default-jdk`, choose the specific modern version you want to target:

```bash
# For Java 17
sudo dnf install java-17-openjdk-devel

# For Java 21
sudo dnf install java-21-openjdk-devel

```

---

## Modern Containers (Docker vs. Podman)

RHEL 10 completely pushes **Podman 5.x** as its native container runtime. It includes a built-in compose tool natively via the package manager.

If you want the RHEL-native, daemonless architecture, install:

```bash
sudo dnf install podman podman-compose

```

*(Note: If your projects strictly require the actual Docker CE daemon backend instead of Podman, Docker's official repository for RHEL 10/CentOS Stream 10 must be added manually to your system before running `dnf install docker-ce docker-compose-plugin`).*