# One Identity Manager API Server

This image runs a One Identity Manager API Server instance. It downloads the necessary files at startup from the configured database. The behavior can be controlled by secret values and environment variables.

## Requirements

All images tagged with `windows-amd64` are compatible with Windows Server hosts. Please check the supported Windows Server version for each tag using the list below.

All images tagged with `linux-amd64` are compatible with Linux OS hosts.

### Supported Windows Server 2025 amd64 tags

* `latest`, `10.0`, `windows-amd64-10.0-windowsservercore-ltsc2025`, `windows-amd64-latest-windowsservercore-ltsc2025`

### Supported Windows Server 2022 amd64 tags

* `latest`, `10.0`, `9.3`, `9.2`, `9.1`, `9.0`, `windows-amd64-10.0-windowsservercore-ltsc2022`, `windows-amd64-9.3-windowsservercore-ltsc2022`, `windows-amd64-9.2-windowsservercore-ltsc2022`, `windows-amd64-9.1-windowsservercore-ltsc2022`, `windows-amd64-9.0-windowsservercore-ltsc2022`, `windows-amd64-latest-windowsservercore-ltsc2022`

### Supported Windows Server 2019 amd64 tags

* `9.3`, `9.2`, `9.1`, `9.0`, `windows-amd64-9.3-windowsservercore-ltsc2019`, `windows-amd64-9.2-windowsservercore-ltsc2019`, `windows-amd64-9.1-windowsservercore-ltsc2019`, `windows-amd64-9.0-windowsservercore-ltsc2019`

### Supported Windows Server 2016 amd64 tags

* `9.2`, `9.1`, `9.0`, `windows-amd64-9.2-windowsservercore-ltsc2016`, `windows-amd64-9.1-windowsservercore-ltsc2016`, `windows-amd64-9.0-windowsservercore-ltsc2016`

### Supported Linux tags

* `latest`, `9.3`, `9.2`, `9.1`, `9.0`, `linux-amd64-9.3`, `linux-amd64-9.2`, `linux-amd64-9.1`, `linux-amd64-9.0`, `linux-amd64-latest`

## How to use this image

> **Disclaimer regarding Linux images**
>
> Please note that not all functions on Identity Manager are supported in Linux containers.
> Identity Manager < 9.3 runs on Mono and not all underlying components are available with Mono.
> For example report generation may fail running in Linux containers.
> 
> Identity Manager >= 9.3 runs on .NET and not all underlying components are compatible with .NET on Linux.

> **Remark:** On Linux the API server image does not update automatically. You have to
> start a new instance when entries of `QBMFileRevision` change by updates or migrations.

### Configuration files

#### `Web.config`

The `Web.config` file configures all settings of the API server. The container probes two locations for a `Web.config` file:

##### Windows

1. `C:\ProgramData\Docker\secrets\Web.config`: If this file is present it will be used without replacing any placeholders with environment variables. A template `Web.config` can be created using this call:

   ```powershell
   $P = $PWD.Path.Replace("\", "/")
   $VERSION = "latest"
   docker run -it --rm `
       -v $P:C:/Data `
       -e "CONNSTRING=Data Source=..." `
       -e "UPDATEPWD=..." `
       -e BASEURL=http://localhost:9999/ `
       -e APPSERVERCONNSTRING=URL=http://localhost/AppServer `
       oneidentity/oneim-api:$VERSION `
       --createwebcfg C:\data
   ```

   Alternatively the connection string can come from a secrets folder and the `web.config` can be put there immediately:

   ```powershell
   $P = $PWD.Path.Replace("\", "/")
   $VERSION = "latest"
   docker run -it --rm `
       -v $P/secrets:C:/ProgramData/Docker/secrets `
       -e BASEURL=http://localhost:9999/ `
       oneidentity/oneim-api:$VERSION `
       --createwebcfg C:/ProgramData/Docker/secrets
   ```

    >**Remark:** You need to specify the same values `UPDATEACCOUNT` and `UPDATEPWD` during generation of the web.config and during the start of the container that uses the created web.config.

2. The container uses it's template and replaces following values with either secret
   values or environment variables:

| Replacement pattern      | Secret value                                         | Environment variable    | Comment                                                                       |
|--------------------------|------------------------------------------------------|-------------------------|-------------------------------------------------------------------------------|
| `#FACTORY#`              | `C:\ProgramData\Docker\secrets\DBSYSTEM`             | `DBSYSTEM`              | Value depends on `DBSYSTEM` value, but contains the concrete DB factory class |
| `#CONNSTR#`              | `C:\ProgramData\Docker\secrets\CONNSTRING`           | `CONNSTRING`            |                                                                               |
| `%HDB_NAME_1%`           | `C:\ProgramData\Docker\secrets\HDBNAME1`             | `HDBNAME1`              |                                                                               |
| `%HDB_CONNECTION_1%`     | `C:\ProgramData\Docker\secrets\HDBCONN1`             | `HDBCONN1`              |                                                                               |
| `%HDB_NAME_2%`           | `C:\ProgramData\Docker\secrets\HDBNAME2`             | `HDBNAME2`              |                                                                               |
| `%HDB_CONNECTION_2%`     | `C:\ProgramData\Docker\secrets\HDBCONN2`             | `HDBCONN2`              |                                                                               |
| `%HDB_NAME_3%`           | `C:\ProgramData\Docker\secrets\HDBNAME3`             | `HDBNAME3`              |                                                                               |
| `%HDB_CONNECTION_3%`     | `C:\ProgramData\Docker\secrets\HDBCONN3`             | `HDBCONN3`              |                                                                               |
| `#TARGETS#`              |                                                      | `TARGETS`               |                                                                               |
| `#BASEURL#`              | `C:\ProgramData\Docker\secrets\BASEURL`              | `BASEURL`               |                                                                               |
| `#ROOTDIR#`              |                                                      | `SRV_HOME`              |                                                                               |
| `#APPSERVERCONNSTRING#`  | `C:\ProgramData\Docker\secrets\APPSERVERCONNSTRING`  | `APPSERVERCONNSTRING`   |                                                                               |
| `#UPDATEUSER#`           | `C:\ProgramData\Docker\secrets\UPDATEUSER`           | `UPDATEUSER`            |                                                                               |
| `#APPLICATIONTOKEN#`     | `C:\ProgramData\Docker\secrets\APPLICATIONTOKEN`     | `APPLICATIONTOKEN`      |                                                                               |
| `#FORCEAPPTOKEN#`        |                                                      | `FORCEAPPTOKEN`         |                                                                               |
| `#APPINSIGHTS_KEY#`      | `C:\ProgramData\Docker\secrets\APPINSIGHTS_KEY`      | `APPINSIGHTS_KEY`       |                                                                               |
| `#APPINSIGHTS_LOGLEVEL#` | `C:\ProgramData\Docker\secrets\APPINSIGHTS_LOGLEVEL` | `APPINSIGHTS_LOGLEVEL`  |                                                                               |
| `#LOG_TENANT#`           | `C:\ProgramData\Docker\secrets\LOG_TENANT`           | `LOG_TENANT`            |                                                                               |
| `#FILE_LOGLEVEL#`        | `C:\ProgramData\Docker\secrets\FILE_LOGLEVEL`        | `FILE_LOGLEVEL`         |                                                                               |
| `#TRUSTEDSOURCEKEY#`     | `C:\ProgramData\Docker\secrets\TRUSTEDSOURCEKEY`     | `TRUSTEDSOURCEKEY`      |                                                                               |
| `#REGISTRATIONUSER#`     | `C:\ProgramData\Docker\secrets\REGISTRATIONUSER`     | `REGISTRATIONUSER`      |                                                                               |
| `#ONELOGINAPI#`          | `C:\ProgramData\Docker\secrets\ONELOGINAPI`          | `ONELOGINAPI`           |                                                                               |
| `#OPENAI_ENDPOINT#`      | `C:\ProgramData\Docker\secrets\OPENAI_ENDPOINT`      | `OPENAI_ENDPOINT`       | OpenAI endpoint URL used by the "OpenAI" connection string                    |
| `#OPENAI_API_KEY#`       | `C:\ProgramData\Docker\secrets\OPENAI_API_KEY`       | `OPENAI_API_KEY`        | OpenAI API key used by the "OpenAI" connection string                         |
| `#SUPPORTREADSCALEOUT#`  | `C:\ProgramData\Docker\secrets\SUPPORTREADSCALEOUT`  | `SUPPORTREADSCALEOUT`   |                                                                               |

   The secret value takes precedence over the environment variable.

##### Linux

1. `/run/secrets/web.config`: If this file is present it will be used without replacing
   any placeholders with environment variables. A template `web.config` can be created
   using this call:

   ```sh
   VERSION=latest
   docker run -it --rm -v $PWD:/data -e "CONNSTRING=Data Source=..." -e BASEURL=http://localhost:9999/ -e APPSERVERCONNSTRING=URL=http://localhost/AppServer oneidentity/oneim-api:linux-amd64-$VERSION --createwebcfg /data
   ```

   Alternatively the connection string can come from a secrets folder and the `web.config` can be put there immediately:

   ```sh
   VERSION=latest
   docker run -it --rm -v $PWD/secrets:/run/secrets -e BASEURL=http://localhost:9999/ oneidentity/oneim-api:linux-amd64-$VERSION --createwebcfg /run/secrets
   ```

2. The container uses it's template and replaces following values with either secret
   values or environment variables:

| Replacement pattern      | Secret value                        | Environment variable   | Comment                                                                       |
|--------------------------|-------------------------------------|------------------------|-------------------------------------------------------------------------------|
| `#FACTORY#`              | `/run/secrets/DBSYSTEM`             | `DBSYSTEM`             | Value depends on `DBSYSTEM` value, but contains the concrete DB factory class |
| `#CONNSTR#`              | `/run/secrets/CONNSTRING`           | `CONNSTRING`           |                                                                               |
| `%HDB_NAME_1%`           | `/run/secrets/HDBNAME1`             | `HDBNAME1`             |                                                                               |
| `%HDB_CONNECTION_1%`     | `/run/secrets/HDBCONN1`             | `HDBCONN1`             |                                                                               |
| `%HDB_NAME_2%`           | `/run/secrets/HDBNAME2`             | `HDBNAME2`             |                                                                               |
| `%HDB_CONNECTION_2%`     | `/run/secrets/HDBCONN2`             | `HDBCONN2`             |                                                                               |
| `%HDB_NAME_3%`           | `/run/secrets/HDBNAME3`             | `HDBNAME3`             |                                                                               |
| `%HDB_CONNECTION_3%`     | `/run/secrets/HDBCONN3`             | `HDBCONN3`             |                                                                               |
| `#TARGETS#`              |                                     | `TARGETS`              |                                                                               |
| `#BASEURL#`              | `/run/secrets/BASEURL`              | `BASEURL`              |                                                                               |
| `#ROOTDIR#`              |                                     | `SRV_HOME`             |                                                                               |
| `#APPSERVERCONNSTRING#`  | `/run/secrets/APPSERVERCONNSTRING`  | `APPSERVERCONNSTRING`  |                                                                               |
| `#UPDATEUSER#`           | `/run/secrets/UPDATEUSER`           | `UPDATEUSER`           |                                                                               |
| `#APPLICATIONTOKEN#`     | `/run/secrets/APPLICATIONTOKEN`     | `APPLICATIONTOKEN`     |                                                                               |
| `#FORCEAPPTOKEN#`        |                                     | `FORCEAPPTOKEN`        |                                                                               |
| `#APPINSIGHTS_KEY#`      | `/run/secrets/APPINSIGHTS_KEY`      | `APPINSIGHTS_KEY`      |                                                                               |
| `#APPINSIGHTS_LOGLEVEL#` | `/run/secrets/APPINSIGHTS_LOGLEVEL` | `APPINSIGHTS_LOGLEVEL` |                                                                               |
| `#LOG_TENANT#`           | `/run/secrets/LOG_TENANT`           | `LOG_TENANT`           |                                                                               |
| `#CONSOLE_LOGLEVEL#`     | `/run/secrets/CONSOLE_LOGLEVEL`     | `CONSOLE_LOGLEVEL`     |                                                                               |
| `#TRUSTEDSOURCEKEY#`     | `/run/secrets/TRUSTEDSOURCEKEY`     | `TRUSTEDSOURCEKEY`     |                                                                               |
| `#REGISTRATIONUSER#`     | `/run/secrets/REGISTRATIONUSER`     | `REGISTRATIONUSER`     |                                                                               |
| `#ONELOGINAPI#`          | `/run/secrets/ONELOGINAPI`          | `ONELOGINAPI`          |                                                                               |
| `#OPENAI_ENDPOINT#`      | `/run/secrets/OPENAI_ENDPOINT`      | `OPENAI_ENDPOINT`      | OpenAI endpoint URL used by the "OpenAI" connection string                    |
| `#OPENAI_API_KEY#`       | `/run/secrets/OPENAI_API_KEY`       | `OPENAI_API_KEY`       | OpenAI API key used by the "OpenAI" connection string                         |
| `#SUPPORTREADSCALEOUT#`  | `/run/secrets/SUPPORTREADSCALEOUT`  | `SUPPORTREADSCALEOUT`  |                                                                               |

   The secret value takes precedence over the environment variable.

#### `Session certificate`

The certificate that is used for encryption of session information will be searched under

* **Windows:** `C:\ProgramData\Docker\secrets\SessionCertificate.pfx`
* **Linux:** `/run/secrets/SessionCertificate.pfx`

The folder

* **Windows:** `C:\ProgramData\Docker\secrets\`
* **Linux:** `/run/secrets`

can be either mounted as volume where an existing certificate by that name can be found, or the service will try to create a new certificate at that location if the folder is writeable.

The session certificate needs to be shared between API server instances, so that sessions can be reopened after API server updates or in clusters of API servers.

##### Windows

A session certificate can be created by this call:

   ```powershell
   $P = $PWD.Path.Replace("\", "/")
   $VERSION = "latest"
   docker run -it --rm -v $P/secrets:C:/Data oneidentity/oneim-api:$VERSION --createcert C:\Data
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
    -Subject "CN=API Server" `
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
docker run -it --rm -v $PWD:/data oneidentity/oneim-api:linux-amd64-$VERSION --createcert /data
```

or by using OpenSSL:

```sh
openssl genrsa 2048 > private.pem
openssl req -x509 -sha1 -days 1000 -new -subj "/CN=API Server" -key private.pem -out public.pem
openssl pkcs12 -export -in public.pem -inkey private.pem -password pass: -out config/SessionCertificate.pfx
```

### Environment variables

#### `DBSYSTEM`

Database or application server system to connect during installation and runtime. Valid values are

* `MSSQL`: Microsoft SQL Server
* `APPSERVER`: Connect using an application server

If no `DBSYSTEM` value is provided, `MSSQL` is being used as default.

Can be provided in the secrets folder.

#### `CONNSTRING`

Connection string to be used to connect to the system. This is a mandatory parameter. The content depends on the used database system.

Samples of connection strings for the different systems are:

`MSSQL`:

```connectionstring
Data Source=[Server];Initial Catalog=[DB];User ID=[User];Password=[Password]
```

`APPSERVER`:

```connectionstring
Url=https://my.app.server/
```

Can be provided in the secrets folder.

#### `APPSERVERCONNSTRING`

Additional Application Server connection string if the main connection is using a direct Microsoft SQL Server connection. This parameter is mandatory if the parameter `DBSYSTEM` equals `MSSQL`.

```connectionstring
Url=https://my.app.server/
```

Can be provided in the secrets folder.

#### `OPENAI_ENDPOINT`

URL of the OpenAI endpoint used by the application. Populates the OpenAI connection string in web.config. Example:

```text
https://models.example.com/openai
```

Can be provided in the secrets folder.

#### `OPENAI_API_KEY`

API key for the OpenAI endpoint. Populates the OpenAI connection string in web.config.

Can be provided in the secrets folder.

#### `TARGETS`

Comma separated list of deployment targets to install. The default target is: `Server\Web\BusinessAPIServer,Server\Web\BusinessApiServer\AppInsights,Server\Web\BusinessApiServer\EndUserApps`.

#### `UPDATEUSER`

Authentication string of a user to be used for auto update handling and for the creation of the `QBMWebApplication` entry during startup.

Can be provided in the secrets folder.

#### `BASEURL`

Identifier for the `QBMWebApplication` entry to create (if not existent) and to use for configuration.

Can be provided in the secrets folder.

#### `UPDATEACCOUNT`

>**Remark:** Windows only. The environment variable is only available in the Windows containers.

Set this value to create a specific account name for updates. The default name of the update account is `Update`.

Can be provided in the secrets folder.

#### `UPDATEPWD`

>**Remark:** Windows only. The environment variable is only available in the Windows containers.

Password for the update account specified in `UPDATEACCOUNT`. If no password is set, a random password will be generated for the container run.

Can be provided in the secrets folder.

#### `HDBNAMEx`

Name of a history database connection to put into the Web.config. x can be any value from 1 to 3. Requires a `HDBCONNx` value.

Can be provided in the secrets folder.

#### `HDBCONNx`

Connection string of a history database connection to put into the Web.config. x can be any value from 1 to 3.

Can be provided in the secrets folder.

#### `APPLICATIONTOKEN`

Sets an application token for the Password Reset Portal. This application token functions like a password, which the web application uses to authenticate itself on the database. The application token is saved as a hash value in the database in the configuration parameter "QER\Person\PasswordResetAuthenticator\ApplicationToken" and stored encrypted in the file web.config.

Can be provided in the secrets folder.

#### `FORCEAPPTOKEN`

Force setting application in the database over existing one. Without this setting an existing application token will not be overwritten.

#### `TRUSTEDSOURCEKEY`

Mark this application as trusted. That lowers some SQL checks for the application and allows it to use some riskier statements like `LIKE`, `<`, or `>`. This value needs to contain a clear text key, similar to a password. The matching hashed value will be set in the corresponding QBMWebApplication in the field TrustedSourceKey, if there is no key set yet.

Can be provided in the secrets folder.

#### `FORCETRUSTEDSOURCEKEY`

If this values is set to `true`, overwrite an existing entry in QBMWebApplication.TrustedSourceKey on startup.

#### `GENERATETRUSTEDSOURCEKEY`

If this values is set to `true`, a new key is generated on startup. This setting implies `FORCETRUSTEDSOURCEKEY`, it always overwrites the existing value.

#### `REGISTRATIONUSER`

Authentication string of a user to be used for new user registration.

Can be provided in the secrets folder.

#### `ONELOGINAPI`

Connection string to the OneLogin API. The API Server uses this connection to communicate with the OneLogin MFA API. This string must have the following format:

```text
Domain=<your-domain>.onelogin.com;ClientId=<client id>;ClientSecret=<client secret>
```

Can be provided in the secrets folder.

#### `SUPPORTREADSCALEOUT`

Set to `true` if the application should use read scale-out handling of Azure Managed Instances.

#### `APPINSIGHTS_KEY`

Key to use for logging to AppInsights. If it is not set no logging to AppInsights will be done.

Can be provided in the secrets folder.

#### `APPINSIGHTS_LOGLEVEL`

Log level to use for logging to AppInsights. Default: `Info`

Can be provided in the secrets folder.

#### `LOG_TENANT`

Tenant ID to be assiociated with every log message sent to AppInsights.

Can be provided in the secrets folder.

#### `CONSOLE_LOGLEVEL`

Log level to use for logging to console. Linux only. Default: `Info`.

Can be provided in the secrets folder.

#### `FILE_LOGLEVEL`

Log level to use for logging to log file. Windows only. Default: `Info`

Can be provided in the secrets folder.

#### `DEBUG`

Set this to `true` to get verbose output from the startup script.

### Volume mount points

The volume mount points differ between the Windows and Linux based images. Please specify the volume points according to the type of image used.

#### `secrets`

* **Windows:** `C:/ProgramData/Docker/secrets`
* **Linux:** `/run/secrets`

It can contain files with the name of the setting and its value as content.

This mount point should be mounted read-only.

#### `ca-certificates`

* **Windows:** `C:/ca-certificates`
* **Linux:** `/run/ca-certificates`

Place custom certificates that should be trusted here. Any certificate with a .crt file extension will be imported to the container (or mono) local machine trusted root certificate store during startup of the container. Alternatively, an archive called `ca-certificates.zip` can also be placed in this directory. This archive will be extracted during startup and all containing .crt files are imported to the store. This may be required when the application is performing HTTPS calls to a server that is using a certificate signed by a private PKI. Like AppServer with enforced HTTPS. The lookup path may be controlled by changing the CA_CERTIFICATE environment variable.

This mount point should be mounted read-only.

>**Remarks:** For Linux the certificates are required to be in base64 PEM format, binary DER format is not supported.

#### `Cache`

* **Windows:** `C:\Cache`
* **Linux:** `/var/www/App_Data/Cache`

Cached data will be put under the directory `Cache`. This directory can be mounted to a temporary file system to avoid writes to the container file system.

#### `C:\Logs`

* **Windows:** `C:\Logs`
* **Linux:** Not available

API Server logs are written into the folder `C:\Logs` in Windows containers. Mount this folder to get access to log files.

Linux based API servers log directly to standard output. These logs can be fetched using `docker logs` or aggregated by a container runtime.

### Ports

The API Server is exposing port **8080**.

## Samples

### Windows sample

Run the container in the background and get parameters from entries in the secrets folder:

```powershell
$P = $PWD.Path.Replace("\", "/")
docker run -d `
    -v $P/secrets:C:/ProgramData/Docker/secrets:ro `
    -v $P/data/cache:C:/Cache `
    -v $P/data/search:C:/Search `
    -v $P/data/logs:C:/Logs `
    -p 9999:8080 `
    -e BASEURL=http://localhost:9999/ `
    oneidentity/oneim-api:latest
```

The secrets folder looks like:

```text
secrets
|-- APPSERVERCONNSTRING
|-- CONNSTRING
|-- DBSYSTEM
|-- ONELOGINAPI
|-- OPENAI_ENDPOINT
|-- OPENAI_API_KEY
|-- REGISTRATIONUSER
|-- UPDATEUSER
`-- UPDATEPWD
```

### Linux sample

Run the container in the background and get parameters from entries in the secrets folder:

```sh
docker run -d \
    -p 9999:8080 \
    -v $PWD/secrets:/run/secrets:ro \
    -e BASEURL=http://localhost:9999/ \
     oneidentity/oneim-api:latest
```

The secrets folder looks like:

```text
secrets/
|-- APPSERVERCONNSTRING
|-- CONNSTRING
|-- DBSYSTEM
|-- ONELOGINAPI
|-- OPENAI_ENDPOINT
|-- OPENAI_API_KEY
|-- REGISTRATIONUSER
`-- UPDATEUSER
```

## Further Reading

You will find more information about authentication strings in One Identity Manager here [One Identity Manager - Initial Data for Authentication Modules](https://support.oneidentity.com/technical-documents/identity-manager/9.2/authorization-and-authentication-guide/26#TOPIC-2083705).

[One Identity Manager Product Information](https://www.oneidentity.com/products/identity-manager/)
