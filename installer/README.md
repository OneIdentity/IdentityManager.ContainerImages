# One Identity Manager installer image

This image contains a simple installer to be used in derived images to create the app file structure of One Identity Manager.

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

This image is not intended to be used directly but contains everything necessary to create One Identity Manager installation folders in a multi-stage build scenario of custom One Identity Manager images.

### `create-web-dir`

The tool `create-web-dir` allows you to assemble One Identity Manager installation folders from a directory containing a One Identity Manager distribution or directly from an installed One Identity Manager database.

* **Windows:** `C:/installer/create-web-dir.exe`
* **Linux:** `dotnet /installer/create-web-dir.dll`

#### Parameters

| Parameter | Description |
| --- | --- |
| --help, -h, -? | Show this help. |
| --mode=VALUE | Mode, allowed: web, standalone, default: web |
| --dest, -d=VALUE | Destination directory |
| --setup, -s=VALUE | Setup directory containing the modules. Provide either this value or a database connection. |
| --db-system=VALUE | Database system to connect as source. Valid values are MSSQL, ORACLE, or APPSERVER. Default: MSSQL |
| --db-connect=VALUE | Database connection string for the system to use as source. |
| --auth=VALUE | Authentication used during installation. Required for some options when running against application servers. |
| --modules, -m=VALUE | Comma separated list of modules to install. May be omitted when installing from an existing database. |
| --targets, -t=VALUE | Comma separated list of deployment targets to install. |
| --appserver-url=VALUE | Connect the resulting app to an AppServer instead of the source database. |
| --web-config, -w=VALUE | Web.config file |
| --nlog-config, -n=VALUE | NLog config file |
| --session-cert, -c=VALUE | Session certificate file |
| --create-session-cert | Create a new session certificate, if the file doesn't exist. |
| --session-cert-issuer=VALUE | Issuer name to use for a generated session certicate (default: "CN=Application Server") |
| --variable, -v=VALUE | Variable to replace (Key=Value) |
| --config-from-server | Get job service config from the server in the database and put it into the destination. Parameter --server-name becomes mandatory when setting this parameter. |
| --targets-from-server | Get targets attached to the server in the database. Parameter --server-name becomes mandatory when setting this parameter. |
| --server-name=VALUE | Server name as set in the QBMServer table of the database. |
| --web-app=VALUE | Create a web application in the database for the supplied URL |
| --web-app-product=VALUE | Identifier or UID of the dialog product to use in the web application. |
| --nologo | Do not show the start banner and copyright info. |

## Examples

Examples of how to assemble an One Identity Manager installation folder using 'create-web-dir' during a custom Docker image build.

### Custom Dockerfile to create a simple Job Service installation (Linux)

The following Dockerfile demonstrates the usage of the `oneidentity/oneim-installer` image to create your own custom One Identity Manager Job Service container for Linux.

```Dockerfile
FROM oneidentity/oneim-installer:linux-amd64-latest

# Database system, supported values: MSSQL
ENV DBSYSTEM MSSQL

# Connection string to the database. Will be used at build.
#
# MS SQL connection string
# ENV CONNSTRING "Data Source=<server>;Initial Catalog=<database>;User ID=<user>;Password=<password>"

# Deployment targets to install
ENV TARGETS "Server\Jobserver"

# Assemble the One Identity Manager installation
RUN dotnet /installer/create-web-dir.dll \
    --mode=standalone \
    --dest=${SRV_HOME} \
    --db-system=$DBSYSTEM \
    --db-connect="$CONNSTRING" \
    --targets="$TARGETS" \
    && rm -rf /installer

# Copy a valid job service configuration file to the created One Identity Manager installation
COPY JobService.cfg ${SRV_HOME}

EXPOSE 1880
EXPOSE 2880

WORKDIR ${SRV_HOME}
CMD ["dotnet", "JobServiceCmd.dll"]
```

## Further Reading
[One Identity Manager Product Information](https://www.oneidentity.com/products/identity-manager/)
