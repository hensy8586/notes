# Database Access

## Access Snowflake Database

A simple way to access Snowflake database locally using Python is as follows:

```python
import snowflake.connector

con = snowflake.connector.connect(
    user="<user-name>",
    account="<snowflake-account>",  # at 3M it's mmm-ww.privatelink
    authenticator="externalbrowser",
    session_parameters={"QUERY_TAG": "my test"}
)
cs = con.cursor()

# query script
content = cs.execute(
    """
    SELECT * FROM table
    """
).fetchall()

### Some additional actions ###

cs.close()
con.close()
```

To get data as a Pandas dataframe, do the following:

```python
import pandas as pd

df = pd.DataFrame(content, columns=[col[0] for col in cs.description])
```

## Access SQL Server database (on Mac)

It's a much simpler process to access SQL Server database on Windows, but on Mac it can be tricky. Below is a method that's proven to work.

First we need a bunch of information for the database:

```python
import getpass

SERVER = "<server>"
PORT = "1433"  # 1433 is often a good starting one
DATABASE = "<database-name>"
DOMAIN = "<domain-name>"
USER = "<username>"
PASSWORD = getpass.getpass("Enter password: ")  # getpass would prompt for user to enter directly
```

Now, we need to go to JTDS to download a driver: [https://jtds.sourceforge.net](https://jtds.sourceforge.net) and keep its directory:

```python
JTDS_JAR = "<directory-of-jar-file>"
```

Now we can query with the code below:

```python
import jaydebeapi
import csv

jdbc_url = (
    f"jdbc:jtds:sqlserver://{SERVER}:{PORT}/{DATABASE};"
    f"useNTLMv2=true;domain={DOMAIN};"
)
query_res_file = "result.csv"

con = jaydebeapi.connect(
    "net.sourceforge.jtds.jdbc.Driver",
    jdbc_url, [USER, PASSWORD], JTDS_JAR
)
cs = con.cursor()
cs.execute(
    """
    SELECT * FROM table
    """
)

# fetch all rows
rows = cs.fetchall()
# get column headers
headers = [desc[0] for desc in cs.description]
# write to csv
with open(query_res_file, "w", newline="") as f:
    writer = csv.writer(f)
    writer.writerow(headers)
    writer.writerows(rows)
```
