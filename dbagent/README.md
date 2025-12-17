# One Identity Manager Database Agent

This image runs an instance of the One Identity Manager Database Agent. It downloads the necessary files at startup. The behavior can be controlled by secret values and environment variables.

## Requirements

All images tagged with `windows-amd64` are compatible with Windows Server hosts. Please check the supported Windows Server version for each tag using the list below.

All images tagged with `linux-amd64` are compatible with Linux OS hosts.

### Supported Windows Server 2022 amd64 tags

* `9.3`, `windows-amd64-9.3-windowsservercore-ltsc2022`

### Supported Windows Server 2019 amd64 tags

* `9.3`, `windows-amd64-9.3-windowsservercore-ltsc2019`

### Supported Linux tags

* `9.3`, `linux-amd64-9.3`

## How to use this image

### Environment variables

#### `CONNSTRING`

Connection string to be used to connect to the MS SQL database. This is a mandatory parameter.

A sample connection string:

```connectionstring
Data Source=[Server];Initial Catalog=[DB];User ID=[User];Password=[Password]
```

This value can also be provided as a file in the secrets folder.

#### `APPINSIGHTS_KEY`

Key to use for logging to AppInsights. If it is not set no logging to AppInsights will be done.

#### `APPINSIGHTS_LOGLEVEL`

Log level to use for logging to AppInsights. Default: `Info`

#### `LOG_TENANT`

Tenant ID to be assiociated with every log message sent to AppInsights.

#### `CONSOLE_LOGLEVEL`

Log level to use for logging to console. Default: `Info`

### Volume mount points

The volume mount points differ between the Windows and Linux based images. Please specify the volume points according to the type of image used.

#### `secrets`

* **Windows:** `C:/ProgramData/Docker/secrets`
* **Linux:** `/run/secrets`

The directory can contain the following settings as files that have their value as content: `CONNSTRING`.

This mount point should be mounted read-only.

## Samples

### Windows

Run the Database Agent against a Microsoft SQL Server database:

```powershell
docker run -it --rm -e "CONNSTRING=Data Source=[Server];Initial Catalog=[DB];User ID=[User];Password=[Password]" oneidentity/oneim-dbagent:latest
```

Run the container in the background and get parameters from entries in the secrets folder:

```powershell
docker run -d -v C:/Path/To/secrets:C:/ProgramData/Docker/secrets:ro oneidentity/oneim-dbagent:latest
```

The secrets folder looks like:

```
secrets
└── CONNSTRING
```

### Linux

Run the Database Agent against a Microsoft SQL Server database:

```sh
docker run -it --rm -e "CONNSTRING=Data Source=[Server];Initial Catalog=[DB];User ID=[User];Password=[Password]" oneidentity/oneim-dbagent:latest
```

Run the container in the background and get parameters from entries in the secrets folder:

```sh
docker run -d -v -v $PWD/secrets:/run/secrets:ro oneidentity/oneim-dbagent:latest
```

The secrets folder looks like:

```
secrets
└── CONNSTRING
```

## Further Reading
[One Identity Manager Product Information](https://www.oneidentity.com/products/identity-manager/)
