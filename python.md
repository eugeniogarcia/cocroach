# Setup

```ps
Start-Job -ScriptBlock {cockroach.exe start-single-node --insecure --http-addr=localhost:8080 --listen-addr=localhost:26257}
```

```ps
receive-job 1

*
    + CategoryInfo          : NotSpecified: (*:String) [], RemoteException
    + FullyQualifiedErrorId : NativeCommandError
    + PSComputerName        : localhost

* WARNING: RUNNING IN INSECURE MODE!
*
* - Your cluster is open for any client that can access localhost.
* - Any user, even root, can log in without providing a password.
* - Any user, connecting as root, can read or write any data in your cluster.
* - There is no network encryption nor authentication, and thus no confidentiality.
*
* Check out how to secure your cluster:
https://www.cockroachlabs.com/docs/v19.2/secure-a-cluster.html
*
CockroachDB node starting at 2020-10-06 05:41:59.4875864 +0000 UTC (took 1.4s)
build:               CCL v19.2.10 @ 2020/08/19 17:29:35 (go1.12.12)
webui:               http://localhost:8080
sql:                 postgresql://root@localhost:26257?sslmode=disable
RPC client flags:    C:\Software\cockroach-v19.2.10.windows-6.2-amd64\cockroach.exe <client cmd> --host=localhost:26257 --insecure
logs:                C:\Users\Eugenio\Documents\cockroach-data\logs
temp dir:            C:\Users\Eugenio\Documents\cockroach-data\cockroach-temp671704971
external I/O path:   C:\Users\Eugenio\Documents\cockroach-data\extern
store[0]:            path=C:\Users\Eugenio\Documents\cockroach-data
status:              restarted pre-existing node
clusterID:           7e9f09a9-708d-4d0d-8f37-6a79cc23bd97
nodeID:              1
```

## Crea la base de datos

En el log de arranque podemos ver la cadena sql:

```txt
postgresql://root@localhost:26257?sslmode=disable
```

La misma para la base de datos `movr` seria:

```txt
postgresql://root@localhost:26257/movr?sslmode=disable
```

Nos conectamos a la consola:

```ps
cockroach sql --url postgresql://root@localhost:26257/movr?sslmode=disable
```

Creamos la base de datos, y importamos los datos usando un csv:

```ps
CREATE DATABASE movr;
USE movr;

IMPORT TABLE vehicles (
    id UUID PRIMARY KEY,
    last_longitude FLOAT8,
    last_latitude FLOAT8,
    battery INT8,
    last_checkin TIMESTAMP,
    in_use BOOL,
    vehicle_type STRING
    )
    CSV DATA (
        'https://cockroach-university-public.s3.amazonaws.com/10000vehicles.csv'
    )
    WITH delimiter = '|';
```

Podemos ver las tablas y los datos:

```ps
SHOW TABLES;

  table_name
+------------+
  vehicles
(1 row)
```

```ps
SELECT * FROM vehicles LIMIT 10;
```

## Usuarios

```ps
CREATE USER app_user WITH PASSWORD 'Cockroach2020!';

GRANT SELECT, INSERT, UPDATE, DELETE ON DATABASE movr TO app_user;

REVOKE SELECT ON DATABASE movr FROM reports_user;
```

## Flask app

Instalamos las dependencias:

```ps
conda activate cocroach

pip install docopt
pip install flask
pip install flask-bootstrap
pip install flask-login
pip install sqlalchemy
pip install sqlalchemy-cockroachdb
pip install psycopg2-binary
pip install geopy
pip install -U python-dotenv
pip install flask-wtf
```

Arrancamos la app:

```ps
conda activate cocroach

python .\server.py run --url postgresql://root@localhost:26257/movr?sslmode=disable
```

## Conexión a la base de datos

Usaremos la librería [SQL Alchemy](https://docs.sqlalchemy.org/en/13/core/engines.html).