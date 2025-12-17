# One Identity Manager Application Server

This image runs a One Identity Manager Application Server instance. It downloads the necessary files at startup from the configured database. The behavior can be controlled by secret values and environment variables.

## Requirements

All images tagged with `windows-amd64` are compatible with Windows Server hosts. Please check the supported Windows Server version for each tag using the list below.

All images tagged with `linux-amd64` are compatible with Linux OS hosts.

### Supported Windows Server 2025 amd64 tags

* `latest`, `10.0`, `windows-amd64-10.0-windowsservercore-ltsc2025`, `windows-amd64-latest-windowsservercore-ltsc2025`

### Supported Windows Server 2022 amd64 tags

* `latest`, `10.0`, `windows-amd64-10.0-windowsservercore-ltsc2022`, `windows-amd64-latest-windowsservercore-ltsc2022`

### Supported Linux tags

* `latest`, `10.0`, `linux-amd64-10.0`, `linux-amd64-latest`

## How to use this image

> **Disclaimer regarding Linux images**
>
> Please note that not all functions on Identity Manager are supported in Linux containers.
> 
> Identity Manager 10.0 runs on .NET 10 and not all underlying components are compatible with .NET 10 on Linux.

> **Remark:** On Linux the application server image does not update automatically. You have to
> start a new instance when entries of `QBMFileRevision` change by updates or migrations.

### Configuration files

#### `appsettings.json`

The `appsettings.json` file configures all settings of the application server. The container probes two locations for a `appsettings.json` file:

##### Windows

1. `C:\ProgramData\Docker\secrets\appsettings.json`: If this file is present it will be used without replacing any placeholders with environment variables. A template `appsettings.json` can be created using this call:

    ```powershell
    $P = $PWD.Path.Replace("\", "/")
    $VERSION = "latest"
    docker run -it --rm `
    -v $P/secrets:C:/Data `
    -e "CONNSTRING=Data Source=..." `
    -e "UPDATEPWD=..." `
    oneidentity/oneim-appserver:$VERSION `
    --createwebcfg C:\data
    ```
    >**Remark:** You need to specify the same values `UPDATEACCOUNT` and `UPDATEPWD` during generation of the web.config and during the start of the container that uses the created web.config.

2. The container uses it's template and replaces following values with either secret values or environment variables:

| Replacement pattern      | Secret value                                         | Environment variable   |
| ------------------------ | ---------------------------------------------------- | ---------------------- |
| `%DB_PROVIDER%`          | `C:\ProgramData\Docker\secrets\DBSYSTEM`             | `DBSYSTEM`             |
| `%DB_CONNECTION%`        | `C:\ProgramData\Docker\secrets\CONNSTRING`           | `CONNSTRING`           |
| `%HDB_NAME_1%`           | `C:\ProgramData\Docker\secrets\HDBNAME1`             | `HDBNAME1`             |
| `%HDB_CONNECTION_1%`     | `C:\ProgramData\Docker\secrets\HDBCONN1`             | `HDBCONN1`             |
| `%HDB_NAME_2%`           | `C:\ProgramData\Docker\secrets\HDBNAME2`             | `HDBNAME2`             |
| `%HDB_CONNECTION_2%`     | `C:\ProgramData\Docker\secrets\HDBCONN2`             | `HDBCONN2`             |
| `%HDB_NAME_3%`           | `C:\ProgramData\Docker\secrets\HDBNAME3`             | `HDBNAME3`             |
| `%HDB_CONNECTION_3%`     | `C:\ProgramData\Docker\secrets\HDBCONN3`             | `HDBCONN3`             |
| `#BASEURL#`              | `C:\ProgramData\Docker\secrets\BASEURL`              | `BASEURL`              |
| `#APPINSIGHTS_KEY#`      | `C:\ProgramData\Docker\secrets\APPINSIGHTS_KEY`      | `APPINSIGHTS_KEY`      |
| `#APPINSIGHTS_LOGLEVEL#` | `C:\ProgramData\Docker\secrets\APPINSIGHTS_LOGLEVEL` | `APPINSIGHTS_LOGLEVEL` |
| `#LOG_TENANT#`           | `C:\ProgramData\Docker\secrets\LOG_TENANT`           | `LOG_TENANT`           |
| `#FILE_LOGLEVEL#`        | `C:\ProgramData\Docker\secrets\FILE_LOGLEVEL`        | `FILE_LOGLEVEL`        |
| `#SUPPORTREADSCALEOUT#`  | `C:\ProgramData\Docker\secrets\SUPPORTREADSCALEOUT`  | `SUPPORTREADSCALEOUT`  |
| `#DISABLE_LOGVIEWER#`    | `C:\ProgramData\Docker\secrets\ENABLE_LOGVIEWER`     | `ENABLE_LOGVIEWER`     |
| `#DISABLE_RESTAPI#`      | `C:\ProgramData\Docker\secrets\ENABLE_RESTAPI`       | `ENABLE_RESTAPI`       |
| `#DISABLE_APPSERVERAPI#` | `C:\ProgramData\Docker\secrets\ENABLE_APPSERVERAPI`  | `ENABLE_APPSERVERAPI`  |
| `#DISABLE_METADATA#`     | `C:\ProgramData\Docker\secrets\ENABLE_METADATA`      | `ENABLE_METADATA`      |
| `#DISABLE_SYNC#`         | `C:\ProgramData\Docker\secrets\ENABLE_SYNC`          | `ENABLE_SYNC`          |

The secret value takes precedence over the environment variable.

##### Linux

1. `/run/secrets/Web.config`: If this file is present it will be used without replacing any placeholders with environment variables. A template `Web.config` can be created using this call:

   ```sh
   VERSION=latest
   docker run -it --rm -v $PWD:/data -e "CONNSTRING=Data Source=..." oneidentity/oneim-appserver:linux-amd64-$VERSION --createwebcfg /data
   ```

2. The container uses it's template and replaces following values with either secret values or environment variables:

| Replacement pattern      | Secret value                        | Environment variable   |
| ------------------------ | ----------------------------------- | ---------------------- |
| `%DB_PROVIDER%`          | `/run/secrets/DBSYSTEM`             | `DBSYSTEM`             |
| `%DB_CONNECTION%`        | `/run/secrets/CONNSTRING`           | `CONNSTRING`           |
| `%HDB_NAME_1%`           | `/run/secrets/HDBNAME1`             | `HDBNAME1`             |
| `%HDB_CONNECTION_1%`     | `/run/secrets/HDBCONN1`             | `HDBCONN1`             |
| `%HDB_NAME_2%`           | `/run/secrets/HDBNAME2`             | `HDBNAME2`             |
| `%HDB_CONNECTION_2%`     | `/run/secrets/HDBCONN2`             | `HDBCONN2`             |
| `%HDB_NAME_3%`           | `/run/secrets/HDBNAME3`             | `HDBNAME3`             |
| `%HDB_CONNECTION_3%`     | `/run/secrets/HDBCONN3`             | `HDBCONN3`             |
| `#BASEURL#`              | `/run/secrets/\BASEURL`             | `BASEURL`              |
| `#APPINSIGHTS_KEY#`      | `/run/secrets/APPINSIGHTS_KEY`      | `APPINSIGHTS_KEY`      |
| `#APPINSIGHTS_LOGLEVEL#` | `/run/secrets/APPINSIGHTS_LOGLEVEL` | `APPINSIGHTS_LOGLEVEL` |
| `#LOG_TENANT#`           | `/run/secrets/LOG_TENANT`           | `LOG_TENANT`           |
| `#CONSOLE_LOGLEVEL#`     | `/run/secrets/CONSOLE_LOGLEVEL`     | `CONSOLE_LOGLEVEL`     |
| `#FILE_LOGLEVEL#`        | `/run/secrets/FILE_LOGLEVEL`        | `FILE_LOGLEVEL`        |
| `#SUPPORTREADSCALEOUT#`  | `/run/secrets/SUPPORTREADSCALEOUT`  | `SUPPORTREADSCALEOUT`  |
| `#DISABLE_LOGVIEWER#`    | `/run/secrets/ENABLE_LOGVIEWER`     | `ENABLE_LOGVIEWER`     |
| `#DISABLE_RESTAPI#`      | `/run/secrets/ENABLE_RESTAPI`       | `ENABLE_RESTAPI`       |
| `#DISABLE_APPSERVERAPI#` | `/run/secrets/ENABLE_APPSERVERAPI`  | `ENABLE_APPSERVERAPI`  |
| `#DISABLE_METADATA#`     | `/run/secrets/ENABLE_METADATA`      | `ENABLE_METADATA`      |
| `#DISABLE_SYNC#`         | `/run/secrets/ENABLE_SYNC`          | `ENABLE_SYNC`          |

The secret value takes precedence over the environment variable.

#### `Session certificate`

The certificate that is used for encryption of session information will be searched under

* **Windows:** `C:\ProgramData\Docker\secrets\SessionCertificate.pfx`
* **Linux:** `/run/secrets/SessionCertificate.pfx`

The folder

* **Windows:** `C:\ProgramData\Docker\secrets\`
* **Linux:** `/run/secrets`

can be either mounted as volume where an existing certificate by that name can be found, or the service will try to create a new certificate at that location if the folder is writeable.

The session certificate needs to be shared between application server instances, so that sessions can be reopened after application server updates or in clusters of application servers.

##### Windows

A session certificate can be created by this call:

   ```powershell
   $P = $PWD.Path.Replace("\", "/")
   $VERSION = "latest"
   docker run -it --rm -v $P/secrets:C:/Data oneidentity/oneim-appserver:$VERSION --createcert C:\Data
   ```

or by using Powershell:

```powershell
$outfile = "Path\to\SessionCertificate.pfx"
$date = (Get-Date).Date.AddDays(1000)
$cert = New-SelfSignedCertificate `
    -KeyAlgorithm RSA `
    -KeyLength 2048 `
    -HashAlgorithm SHA1 `
    -KeyExportPolicy Exportable `
    -NotAfter $date `
    -Subject "CN=Application Server" `
    -Type DocumentEncryptionCert
[System.IO.File]::WriteAllBytes( `
    $outfile, `
    $cert.Export( `
        [System.Security.Cryptography.X509Certificates.X509ContentType]::Pkcs12, `
        $null))
```

##### Linux

A session certificate can be created by this call:

```sh
VERSION=latest
docker run -it --rm -v $PWD:/data oneidentity/oneim-appserver:linux-amd64-$VERSION --createcert /data
```

or by using OpenSSL:

```sh
openssl genrsa 2048 > private.pem
openssl req -x509 -sha1 -days 1000 -new -subj "/CN=Application Server" -key private.pem -out public.pem
openssl pkcs12 -export -in public.pem -inkey private.pem -password pass: -out config/SessionCertificate.pfx
```

### Environment variables

#### `DBSYSTEM`

Database system to connect during installation and runtime. Valid values are

* `MSSQL`: Microsoft SQL Server

If no `DBSYSTEM` value is provided, `MSSQL` is being used as default.

This value can also be provided as a file in the secrets folder.

#### `CONNSTRING`

Connection string to be used to connect to the system. This is a mandatory parameter.

A samples of a connection string is:

`MSSQL`:

```connectionstring
Data Source=[Server];Initial Catalog=[DB];User ID=[User];Password=[Password]
```

This value can also be provided as a file in the secrets folder.

#### `TARGETS`

Comma-separated list of deployment targets to install. Default targets include the application server and search index targets: `Server\Web\AppServer,Server\Web\AppServer\SearchCrawler,Server\Web\AppServer\SearchIndex`.

You can change this variable to `Server\Web\AppServer` if no search index is required.

#### `BASEURL`

Base URL under which the server can be reached from outside. Needs to be set when the API test frontend should be functional.

This value can also be provided as a file in the secrets folder.

#### `UPDATEACCOUNT`

>**Remark:** Windows only. The environment variable is only available in the Windows containers.

Set this value to create a specific account name for updates. The default name of the update account is `Update`.

This value can also be provided as a file in the secrets folder.

#### `UPDATEPWD`

>**Remark:** Windows only. The environment variable is only available in the Windows containers.

Password for the update account specified in `UPDATEACCOUNT`. If no password is set, a random password will be generated for the container run.

This value can also be provided as a file in the secrets folder.

#### `HDBNAMEx`

Name of a history database connection to put into the Web.config. x can be any value from 1 to 3. Requires a `HDBCONNx` value.

This value can also be provided as file in the secrets folder.

#### `HDBCONNx`

Connection string of a history database connection to put into the Web.config. x can be any value from 1 to 3.

This value can also be provided as file in the secrets folder.

#### `SUPPORTREADSCALEOUT`

Set to `true` if the application should use read scale-out handling of Azure Managed Instances.

#### `APPINSIGHTS_KEY`

Key to use for logging to AppInsights. If it is not set no logging to AppInsights will be done.

This value can also be provided as file in the secrets folder.

#### `APPINSIGHTS_LOGLEVEL`

Log level to use for logging to AppInsights. Default: `Info`

This value can also be provided as file in the secrets folder.

#### `LOG_TENANT`

Tenant ID to be assiociated with every log message sent to AppInsights.

This value can also be provided as file in the secrets folder.

#### `CONSOLE_LOGLEVEL`

Log level to use for logging to console. Linux only. Default: `Info`.

This value can also be provided as file in the secrets folder.

#### `FILE_LOGLEVEL`

Log level to use for logging to log file. Default: `Info`

This value can also be provided as file in the secrets folder.

#### `ENABLE_LOGVIEWER`

Enable the log viewer in the status frontend. Valid values: `true` and `false`.
Default: `true`.

#### `ENABLE_RESTAPI`

Enable the REST API. Valid values: `true` and `false`.
Default: `true`.

#### `ENABLE_APPSERVERAPI`

Enable the AppServer API that allows API access for Identity Manager frontend
and tools. Valid values: `true` and `false`. Default: `true`.

#### `ENABLE_METADATA`

Enable the meta data API needed for synchronizations against the AppServer.
Valid values: `true` and `false`. Default: `false`.

#### `ENABLE_SYNC`

Enable the synchronization API needed for synchronizations against the AppServer.
Valid values: `true` and `false`. Default: `false`.

#### `DEBUG`

Set this to `true` to get verbose output from the startup script.

### Volume mount points

The volume mount points differ between the Windows and Linux based images. Please specify the volume points according to the type of image used.

#### `secrets`

* **Windows:** `C:/ProgramData/Docker/secrets`
* **Linux:** `/run/secrets`

The directory `secrets` should contain the `SessionCertificate.pfx` file to allow sessions to be passed from one server to another.

It can contain files with the name of the setting and its value as content.

This mount point should be mounted read-only.

#### `ca-certificates`

* **Windows:** `C:/ca-certificates`
* **Linux:** `/run/ca-certificates`

Place custom certificates that should be trusted here. Any certificate with a .crt file extension will be imported to the container (or mono) local machine trusted root certificate store during startup of the container. Alternatively, an archive called `ca-certificates.zip` can also be placed in this directory. This archive will be extracted during startup and all containing .crt files are imported to the store. This may be required when the application is performing HTTPS calls to a server that is using a certificate signed by a private PKI. The lookup path may be controlled by changing the CA_CERTIFICATE environment variable.

This mount point should be mounted read-only.

>**Remarks:** For Linux the certificates are required to be in base64 PEM format, binary DER format is not supported.

#### `search`

* **Windows:** `C:/Search`
* **Linux:** `/var/search`

The directory `search` contains the search index files. Mount this to a volume or a host directory to avoid a complete rebuild on every container start.

#### `Cache`

* **Windows:** `C:\Cache`
* **Linux:** `/var/www/App_Data/Cache`

Cached data will be put under the directory `Cache`. This directory can be mounted to a temporary file system to avoid writes to the container file system.

#### `Logs`

* **Windows:** `C:\Logs`
* **Linux:** `/var/log/appserver`

Application server logs are written into the folder `C:\Logs` in Windows containers and `/var/log/appserver` in Linux containers. Mount this folder to get access to log files.
Log files are not written when the `FILE_LOGLEVEL` is set to `Off`.

Linux based application servers log directly to standard output. These logs can be fetched using `docker logs` or aggregated by a container runtime.

### Ports

The Application Server is exposing port **8080**.

## Samples

### Windows

Run the container in the background and get parameters from entries in the secrets folder:

```powershell
$P = $PWD.Path.Replace("\", "/")
docker run -d `
    -v $P/secrets:C:/ProgramData/Docker/secrets:ro `
    -v $P/data/cache:C:/Cache `
    -v $P/data/search:C:/Search `
    -v $P/data/logs:C:/Logs `
    -p 8080:8080 `
    oneidentity/oneim-appserver:latest
```

The secrets folder looks like:

```
secrets
├── CONNSTRING
└── SessionCertificate.pfx
```

### Linux

Run the container in the background and get parameters from entries in the secrets folder:

```sh
docker run -d \
    -p 8080:8080 \
    -v $PWD/secrets:/run/secrets:ro \
    -v $PWD/search:/var/search \
     oneidentity/oneim-appserver:latest
```

The secrets folder looks like:

```
secrets
├── CONNSTRING
└── SessionCertificate.pfx
```

## Further Reading

[One Identity Manager Product Information](https://www.oneidentity.com/products/identity-manager/)
