Postlite
========

Postlite is a networked version of SQLite databases that implemented the
Postgres wire protocol. This allows GUI tools to be used on remote SQLite
databases which can make administration easier.

The project works by translating Postgres frontend wire messages into SQLite
transactions and converting results back into Postgres response wire messages.
Many Postgres clients also inspect the `pg_catalog` to determine system
information so Postlite mirrors this catalog by using an attached in-memory
database with virtual tables. The proxy also performs minor rewriting on these
system queries to convert them to usable SQLite syntax.

_Note: This software was a proof of concept of wrapping SQLite and testing with 
the Postgres wire protocol. Currently psql and pg8000 works for the simplest case. 
Connection attempts with psycopg2 or psycopg3 will fail due to missing certain
PostgreSQL features expected by these drivers and libraries. You are welcome 
to submit pull requests or the expectation should be very little progress 
as a occassional hobby project. 

## Usage

To use Postlite, execute the command with the directory that contains your
SQLite databases:

```sh
$ postlite -data-dir /data
```

On another machine, you can connect via the regular Postgres port of 5432:

```sh
$ psql --host HOSTNAME my.db
```

This will connect you to a SQLite database at the path `/data/my.db`.

Use
```
$ postlite --help
```
for commandline usage including how to run at different interface:port.


## Development

Postlite uses virtual tables to simulate the `pg_catalog` so you will need to
enable the `vtable` tag when building or installing.

Uder Windows, the code compiles and runs with msys2-mingw64.

```sh
$ go install -tags vtable ./cmd/postlite
```

Sample python code with server running at port 5433
```
import pg8000

with pg8000.connect(host="127.0.0.1", port="5432", ssl_context=None, 
                      database="my_database", user = ''
                     ) as connection:
    connection.autocommit = True
    
    with connection.cursor() as cursor:
        cursor.execute(
        """
        SELECT version();
        """
        )        
        # Fetch all the rows
        results = cursor.fetchall()
        for row in results:
            for column in row:
                print(str(column) + '\n')

```

With Jupyter notebook and jupysql or ipython-sql

```
%sql postgresql+pg8000://any_user_name:any_password@localhost:5433/my_database
```
works. The database will be created the first time it is accessed.
