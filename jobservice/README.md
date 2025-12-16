# One Identity Manager Service

This image runs an instance of a One Identity Manager Service. It downloads the necessary files for a specific job server at startup. The behavior can be controlled by secret values and environment variables.

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
> For example report generation and on-prem target synchronization may fail running in Linux containers.
>
> Identity Manager >= 9.3 runs on .NET and not all underlying components are compatible with .NET on Linux.
> For example on-prem target synchronization may fail running in Linux containers.

### Environment variables

#### `DBSYSTEM`

Database or application server system to connect during installation and runtime. Valid values are

* `MSSQL`: Microsoft SQL Server
* `APPSERVER`: Connect using an application server

If no `DBSYSTEM` value is provided, `MSSQL` is being used as default.

This value can also be provided as a file in the secrets folder.

#### `CONNSTRING`

Connection string to be used to connect to the system. This is a mandatory parameter. The content depends on the used database system.

Samples of connection strings for the different systems are:

`MSSQL`:

```connectionstring
Data Source=[Server];Initial Catalog=[DB];User ID=[User];Password=[Password]
```

`APPSERVER`:

```connectionstring
Url=https://servername/
```

This value can also be provided as a file in the secrets folder.

#### `RUNDBAGENT`:

Run database agent in this job server. Only valid with `DBSYSTEM` = `MSSQL`. The value can either be empty, `true`, or `false`.

#### `AUTHSTRING`

The user needs to be authenticated when running against an application server. This value is mandatory when `DBSYSTEM` is set to `APPSERVER`. Any valid One Identity Manager authentication string can be used.

Sample:

```connectionstring
Module=DialogUser;User=[User name];Password=[Password]
```

This value can also be provided as a file in the secrets folder.

#### `SERVERNAME`

A mandatory parameter that is used to find the matching `QBMServer` entry in the connected system. This defines the behavior of the started service regarding used deployment targets and the queue it is working on.

This value can also be provided as a file in the secrets folder.

#### `QUEUE`

Set a queue name, if it is different from the server name. The default is
`\\${SERVERNAME}`.

#### `CONFIGFROMDB`

By default, the container generates a configuration for the Job server that is based on the values provided and matches the configured system.

If you want to override this behavior, set the `CONFIGFROMDB` environment variable. This means the configuration data for the Job server is loaded from the database.

**Attention:** In this case, you have full responsibility for configuring the connection strings, private key paths, logging and so on. Ensure that the Job server configuration exists in the database. For example, use the Job Server Editor in the Designer to set up the configuration and save it in the database.

You can use the following placeholders in the configuration: `#CONNSTRING#`, `#AUTHSTRING#`, `#DEBUGMODE#`, `#HTTP_USER#`, `#HTTP_PWD#`, `#REQUESTINTERVAL#`, `#SECRETS#`. They are then replaced during startup.

#### `HTTP_USER`

The status web frontend is protected by a user/password combination. This config option sets the username.

If no username is provided the container generates a random username and provides it in the log at startup.

This value can also be provided as a file in the secrets folder.

#### `HTTP_PWD`

The status web frontend is protected by a user/password combination. This config option sets the password.

If no password is provided the container generates a random password and provides it in the log at startup.

This value can also be provided as a file in the secrets folder.

#### `DEBUGMODE`

Set this variable to `True` to enable debug mode in the service.

#### `REQUESTINTERVAL`

Set the job request interval in seconds. The default value of this parameter is 90 seconds.

#### `EXTERNALSLOTS`

Number of slots running jobs out-of-process. Default: 10

#### `EXTERNALSLOTS32`

Number of slots running jobs out-of-process in a 32 bit process. Default: 4

#### `INTERNALSLOTS`

Number of slots running jobs in-process. Default: 20

#### `REQUESTQUEUELIMIT`

Maximum number of process requests waiting to be handled by the database. Default: 1000

#### `RESULTQUEUELIMIT`

Maximum number of process results waiting to be handled by the database. Default: 10000

#### `FLUSHTIMEOUTSECONDS`

Number of seconds the service is allowed to write back pending results before shutting down. Default: 10

#### `SUPPORTREADSCALEOUT`

Set to `true` if the service should use read scale-out handling of Azure Managed Instances.

#### `APPINSIGHTS_KEY`

Key to use for logging to AppInsights. If it is not set no logging to AppInsights will be done.

#### `APPINSIGHTS_LOGLEVEL`

Log level to use for logging to AppInsights. Default: `Info`

#### `LOG_TENANT`

Tenant ID to be assiociated with every log message sent to AppInsights.

#### `CONSOLE_LOGLEVEL`

Log level to use for logging to console. Default: `Info`

#### `FILE_LOGLEVEL`

Log level to use for raw logs into log file. Default: `Off`

#### `SESSIONAUTH`

Use the Identity Manager authentication modules to authenticate the user against the JobService web frontend. Set it to `true` to switch
on session based authentication.

#### `BASEURL`

Set the base URL to find the right web application configuration. This is needed if the user should be
authentified by OAuth2 or OpenID Connect.

Only valid for session based authentication with `SESSIONAUTH`.

#### `ALLOW_SECRETS_FOR_REPLACEMENT`

Comma separated list of secret names that are allowed for replacement in job
parameters.

#### `REMOTECONNECT_SECRET`

If this value is set the remote connect plugin for the synchronization engine will be installed and initialized with this secret as authentication method.

### Volume mount points

The volume mount points differ between the Windows and Linux based images. Please specify the volume points according to the type of image used.

#### `secrets`

* **Windows:** `C:/ProgramData/Docker/secrets`
* **Linux:** `/run/secrets`

The directory `secrets` should contain the `private.key` file to decrypt any values, that are stored encrypted in the system.

It can contain the following settings as files that have their value as content: `DBSYSTEM`, `CONNSTRING`, `AUTHSTRING`, `SERVERNAME`, `HTTP_USER`, `HTTP_PWD`.

This mount point should be mounted read-only.

#### `ca-certificates`

* **Windows:** `C:/ca-certificates`
* **Linux:** `/run/ca-certificates`

Place custom certificates that should be trusted here. Any certificate with a .crt file extension will be imported to the container (or mono) local machine trusted root certificate store during startup of the container.

Alternatively, an archive called "ca-certificates.zip" can also be placed in this directory. This archive will be extracted during startup and all containing .crt files are imported to the store.

This may be required when the application is performing HTTPS calls to a server that is using a certificate signed by a private PKI. Like AppServer with enforced HTTPS.

The lookup path may be controlled by changing the CA_CERTIFICATE environment variable.

This mount point should be mounted read-only.

>**Remarks:** For Linux the certificates are required to be in base64 PEM format, binary DER format is not supported.

#### `logs`

* **Windows:** `C:/Logs`
* **Linux:** `/var/log/jobservice`

This directory should be mounted to a volume if the service logs should be persisted between runs. These are the log files that the service shows in its web interface.

Additionally the service logs to standard output, these logs can be fetched using `docker logs` or aggregated by a container runtime.

#### `results`

* **Windows:** Not available
* **Linux:** `root/.oneim/AppData/JobService`

This directory should be mounted to a volume, to allow the service to persist pending results between runs. Pending results are the results that have not been signaled to the job queue when the container will be stopped.

>**Remarks:** Due to some issues in the official Windows Server versions (2016 LTSC and 1709), a graceful shutdown of the Job Service container is not supported, therefore pending results will not be persisted. Graceful shutdown of Windows containers running the Job Service is available starting with Windows Server, version 1803.

### Ports

The port **1880** is been used by the service to show it's status frontend. It is protected by the account and password provided in the environment variables `HTTP_USER` and `HTTP_PWD`.
The port **2880** is been used by the remote connect plugin for the synchronization engine. The plugin is used if the environment variable `REMOTECONNECT_SECRET` is set.

## Samples

### Windows

Run the Jobserver for the `QBMServer` entry with identifier `DockerServer1` against a Microsoft SQL Server database:

```powershell
docker run -it --rm `
    -p 1880:1880 `
    -p 2880:2880 `
    -e "CONNSTRING=Data Source=[Server];Initial Catalog=[DB];User ID=[User];Password=[Password]" `
    -e "SERVERNAME=DockerServer1" `
    -e "HTTP_USER=admin" `
    -e "HTTP_PWD=fg(&R8f&Rf" `
    -v C:/Path/To/secrets:C:/ProgramData/Docker/secrets:ro `
    -v C:/Path/To/logs:C:/logs `
    oneidentity/oneim-job:latest
```

Run the container in the background and get parameters from entries in the secrets folder:

```powershell
docker run -d `
    -p 1880:1880 `
    -p 2880:2880 `
    -v C:/Path/To/secrets:C:/ProgramData/Docker/secrets:ro `
    -v C:/Path/To/logs:C:/logs `
    oneidentity/oneim-job:latest
```

The secrets folder looks like:

```
secrets
├── CONNSTRING
├── HTTP_PWD
├── HTTP_USER
├── private.key
└── SERVERNAME
```

Run the service for the `QBMServer` entry with identifier `Docker42` against an application server:

```powershell
docker run -it --rm `
    -p 1880:1880 `
    -p 2880:2880 `
    -e "DBSYSTEM=APPSERVER" `
    -e "CONNSTRING=Url=https://appserver/" `
    -e "AUTHSTRING=Module=DialogUser;User=[Username];Password=[Password]" `
    -e "SERVERNAME=Docker42" `
    -e "HTTP_USER=admin" `
    -e "HTTP_PWD=fg(&R8f&Rf" `
    -v C:/Path/To/secrets:C:/ProgramData/Docker/secrets:ro `
    -v C:/Path/To/logs:C:/logs `
    oneidentity/oneim-job:latest
```

### Linux

Run the service for the `QBMServer` entry with identifier `DockerServer1` against a Microsoft SQL Server database:

```sh
docker run -it --rm \
    -p 1880:1880 \
    -e "CONNSTRING=Data Source=[Server];Initial Catalog=[DB];User ID=[User];Password=[Password]" \
    -e "SERVERNAME=DockerServer1" \
    -e "HTTP_USER=admin" \
    -e "HTTP_PWD=fg(&R8f&Rf" \
    -v $PWD/secrets:/run/secrets:ro \
    -v $PWD/logs:/var/log/jobservice \
    oneidentity/oneim-job:latest
```

Run the container in the background and get parameters from entries in the secrets folder:

```sh
docker run -d \
    -p 1880:1880 \
    -v $PWD/secrets:/run/secrets:ro \
    -v $PWD/logs:/var/log/jobservice \
     oneidentity/oneim-job:latest
```

The secrets folder looks like:

```
secrets
├── CONNSTRING
├── HTTP_PWD
├── HTTP_USER
├── private.key
└── SERVERNAME
```

Run the service for the `QBMServer` entry with identifier `Docker42` against an application server:

```sh
docker run -it --rm \
    -p 1880:1880 \
    -e "DBSYSTEM=APPSERVER" \
    -e "CONNSTRING=Url=https://appserver/" \
    -e "AUTHSTRING=Module=DialogUser;User=[Username];Password=[Password]" \
    -e "SERVERNAME=Docker42" \
    -e "HTTP_USER=admin" \
    -e "HTTP_PWD=fg(&R8f&Rf" \
    -v $PWD/secrets:/run/secrets:ro \
    -v $PWD/logs:/var/log/jobservice \
     oneidentity/oneim-job:latest
```

## Further Reading
[One Identity Manager Product Information](https://www.oneidentity.com/products/identity-manager/)
