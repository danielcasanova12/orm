# SQLite ORM for Neve

A lightweight SQLite query layer written in Neve. It provides `Database`,
`Schema`, `Table`, `Record`, `Query` and `Rows` for schema creation, CRUD,
parameterized filters, typed results and joins.

## Install

With Neve, nsm and the SQLite runtime already installed:

```bash
nsm i danielcasanova12/orm
```

The package uses [db](https://github.com/danielcasanova12/db), listed in
`deps.txt`. The ORM itself has no native `lib.so`. Some versions of nsm report
a 404 while checking for an optional ORM binary; `include.nv` is the package.
The `db` dependency does need its own `lib.so`.

For a clean Ubuntu 22.04 x86-64 installation, follow the
[db setup instructions](https://github.com/danielcasanova12/db#installation-from-scratch)
and then install the ORM. The published native `db` binary uses the Neve LLVM 21 ABI.

## Run the example

```bash
export NEVE_LIBS="$HOME/.local/neve/lib"
neve "$NEVE_LIBS/orm/examples/crud.nv"
neve "$NEVE_LIBS/orm/tests/smoke.nv"
```

Both programs use an in-memory database. The smoke test prints `PACKAGE orm PASS`.

## Usage

```text
import orm

main
    Database database = new Database(":memory:")
    Schema schema = new Schema()
    schema.id("id").text("name").integer("age").real("balance")
    int rc = database.create_table("customers", schema)
    if rc==0
        Table customers = database.table("customers")
        Record customer = new Record()
        customer.text("name", "Daniel").integer("age", 30).real("balance", 42.5)
        rc = customers.insert(customer)
        if rc==0
            Query query = customers.where_text("name", "Daniel")
            Rows rows = query.select()
            while rows.next()==1
                print(rows.text("name"), " / ", rows.real("balance"))
            rows.close()
            if rows.has_error()==1
                print(database.error())
    if rc!=0
        print(database.error())
    database.close()
```

Use a file path instead of `:memory:` for persistence. Values are bound as
parameters; table and column identifiers are quoted. Use `Rows.is_null` to
distinguish NULL from zero or empty text. Close results before closing the
database. The Neve integer and float bindings use 32-bit values.

The example under `examples/` covers insert, typed SELECT, INNER JOIN, update
and delete. Raw SQL and prepared statements remain available through `db`.
