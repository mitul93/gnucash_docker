# GnuCash Container

This project provides a **container to run GnuCash** (GNU Accounting Software) using Ubuntu package.

>Docker may work but is not tested. This project is developed and tested with Podman


It is useful for isolating GnuCash from your host system, managing dependencies, and running it safely in containers.

---

## Features

- Installs **GnuCash** using Ubuntu’s official packages.
- Prepares the **home directories** (`~/.config`, `~/.share`, etc.) to prevent GTK/dconf warnings.
- Fixes common GUI issues:
  - `dconf-WARNING: failed to commit changes`
  - `MIT-SHM` errors
  - Missing GTK icons or SVGs

## Tested with

- podman Compose 1.0.6
- podman 4.9.3
- Make 4.3 (if want to launch program via make)
- Host OS : Ubuntu 24.04 6.8.0-106-generic
- GnuCash 5.5 Build ID: 5.5+(2023-12-16)

## Data Mapping / Persistent Storage

> Use **~/gnucash_user_data** container directory to save GNU Cash user data.  
> This ensures that all of your accounting data persists between container runs.

Following settings ensures persistent storage of application and user data. By default, GnuCash inside the container stores:

| Host Path                     | Container Path            | Type              |
|-------------------------------|---------------------------|:-----------------:|
| `storage/share/gnucash/`      | ~/.local/share/gnucash/   | Application Data  |
| `storage/config/gnucash`      | ~/.config/gnucash         | Application Data  |
| `storage/config/dconf`        | ~/.config/dconf           | GSettings         |
| `storage/gnucash_user_data`   | ~/gnucash_user_data       | User Data         |

You can override these mappings in `docker-compose.yaml` file.

> More information https://wiki.gnucash.org/wiki/Configuration_Locations#System-wide

The container will run as local user UID:GID with username `gnucash`. Make sure you have permissions to read/write `gnucash_container/storage` directory.

## GNU Cash storage backend

GNU Cash supports **XML**, **SQLite**, **MySQL** and **PostgreSQL** as storage backend. 

> More information https://www.gnucash.org/docs/v5/C/gnucash-guide/basics-files1.html

This project configures GNU Cash with SQLite.

If you want to use another storage backend, update [Dockerfile](Dockerfile) apt package install section.

Example values
```
libdbd-sqlite3
libdbd-mysql
libdbd-pgsql
```
## Build and run container

```
cd gnucash_container
make run_gnucash
```

## Use pre-built docker image

```
podman pull ghcr.io/mitul93/gnucash_container:latest
```

## Verifying Container Images using cosign

```
podman run --rm ghcr.io/sigstore/cosign/cosign:latest \
  verify ghcr.io/mitul93/gnucash_container:latest \
  --certificate-identity "https://github.com/mitul93/gnucash_container/.github/workflows/docker-publish-manual.yml@refs/heads/main" \
  --certificate-oidc-issuer "https://token.actions.githubusercontent.com"
```

## Troubleshooting

### Inconsistency in container run
- Remove `.evn` file and run `make run_gnucash`

### Mount folder ownership problem
- Change ownership of `storage` folder to UID/GID of podman container user
