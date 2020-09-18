---
title: Build a Python App with CockroachDB and psycopg2
summary: Learn how to use CockroachDB from a simple Python application with the psycopg2 driver.
toc: true
twitter: false
---

<div class="filters filters-big clearfix">
    <a href="build-a-python-app-with-cockroachdb.html"><button style="width: 22%" class="filter-button current">Use <strong>psycopg2</strong></button></a>
    <a href="build-a-python-app-with-cockroachdb-sqlalchemy.html"><button style="width: 22%" class="filter-button">Use <strong>SQLAlchemy</strong></button></a>
    <a href="build-a-python-app-with-cockroachdb-django.html"><button style="width: 22%" class="filter-button">Use <strong>Django</strong></button></a>
    <a href="build-a-python-app-with-cockroachdb-pony.html"><button style="width: 22%" class="filter-button">Use <strong>PonyORM</strong></button></a>
    <a href="http://docs.peewee-orm.com/en/latest/peewee/playhouse.html#cockroach-database"><button style="width: 22%" class="filter-button">Use <strong>peewee</strong></button></a>
</div>

This tutorial shows you how build a simple Python application with CockroachDB and the psycopg2 driver. For the CockroachDB back-end, you'll use either a temporary local cluster or a free cluster on CockroachCloud.

## Step 1. Install the psycopg2 driver

To install the Python psycopg2 driver, run the following command:

{% include copy-clipboard.html %}
~~~ shell
$ pip install psycopg2
~~~

For other ways to install psycopg2, see the [official documentation](http://initd.org/psycopg/docs/install.html).

## Step 2. Start CockroachDB

Choose whether to run a temporary local cluster or a free CockroachDB cluster on CockroachCloud. The instructions below will adjust accordingly.

<div class="filters clearfix">
  <button class="filter-button page-level" data-scope="local">Use a Local Cluster</button>
  <button class="filter-button page-level" data-scope="cockroachcloud">Use CockroachCloud</button>
</div>
<p></p>

<section class="filter-content" markdown="1" data-scope="local">

1. If you haven't already, [download the CockroachDB binary](install-cockroachdb.html).
1. Run the [`cockroach demo`](cockroach-demo.html) command:

    {% include copy-clipboard.html %}
    ~~~ shell
    $ cockroach demo \
    --insecure=false \
    --empty
    ~~~

    This starts a temporary, in-memory cluster and opens an interactive SQL shell to the cluster.
1. Take take note of the `(sql/tcp)` connection string in the SQL shell welcome text:

    ~~~
    # Connection parameters:
    #   (console) http://127.0.0.1:61009
    #   (sql)     postgres://root:admin@?host=%2Fvar%2Ffolders%2Fk1%2Fr048yqpd7_9337rgxm9vb_gw0000gn%2FT%2Fdemo255013852&port=26257
    #   (sql/tcp) postgres://root:admin@127.0.0.1:61011?sslmode=require    
    ~~~

    You will use it in your application code later.

</section>

<section class="filter-content" markdown="1" data-scope="cockroachcloud">

### Create a free cluster

1. If you haven't already, [sign up for a CockroachCloud account](https://cockroachlabs.cloud/signup).
1. [Log in](https://cockroachlabs.cloud/) to your CockroachCloud account.
1. On the **Overview** page, click **Create Cluster**.
1. On the **Create new cluster** page, for **Cloud provider**, select **Google Cloud**.
1. For **Regions & nodes**, use the default selection of `California (us-west)` region and 1 node.
1. For **Hardware per node**, select `Option 1` (2vCPU, 60 GB disk).
1. Name the cluster. The cluster name must be 6-20 characters in length, and can include lowercase letters, numbers, and dashes (but no leading or trailing dashes).
1. Click **Next**.
1. On the **Summary** page, enter your credit card details.

    {{site.data.alerts.callout_info}}
    You won't be charged until after your free trial expires in 30 days.
    {{site.data.alerts.end}}

1. In the **Trial Code** field, enter `CRDB30`. Click **Apply**.
1. Click **Create cluster**.

Your cluster will be created in approximately 20-30 minutes. Watch this [Getting Started with CockroachCloud](https://youtu.be/3hxSBeE-1tM) video while you wait.

Once your cluster is created, you will be redirected to the **Cluster Overview** page.

### Create a SQL user

1. In the left navigation bar, click **SQL Users**.
1. Click **Add User**. The **Add User** modal displays.
1. Enter a **Username** and **Password**.
1. Click **Save**.

### Authorize your network

1. In the left navigation bar, click **Networking**.
1. Click **Add Network**. The **Add Network** modal displays.
1. From the **Network** dropdown, select **Current Network** to auto-populate your local machine's IP address.
1. To allow the network to access the cluster's Admin UI and to use the CockroachDB client to access the databases, select the **Admin UI to monitor the cluster** and **CockroachDB Client to access the databases** checkboxes.
1. Click **Apply**.

### Get the connection string

1. In the top-right corner of the Console, click the **Connect** button. The **Connect** modal displays.
1. From the **User** dropdown, select the SQL user you created [earlier](#create-a-sql-user).
1. Verify that the `us-west2 GCP` region and `default_db` database are selected.
1. Click **Continue**. The **Connect** tab is displayed.
1. Click **Connection string** to get the connection string for your cluster.
1. Create a `certs` directory on your local workstation.
1. Click the name of the `ca.crt` file to download the CA certificate to your local machine.
1. Move the downloaded `ca.crt` file to the `certs` directory.

</section>

## Step 3. Create a database

<section class="filter-content" markdown="1" data-scope="local">

In the SQL shell, create the `bank` database that your application will use:

{% include copy-clipboard.html %}
~~~ sql
> CREATE DATABASE bank;
~~~

</section>

<section class="filter-content" markdown="1" data-scope="cockroachcloud">

1. If you haven't already, [download the CockroachDB binary](install-cockroachdb.html).
1. Start the [built-in SQL shell](cockroach-sql.html):

    {% include copy-clipboard.html %}
    ~~~ shell
    $ cockroach sql \
    --url='postgres://<username>:<password>@<global host>:26257?sslmode=verify-full&sslrootcert=<certs_dir>/<ca.crt>'
    ~~~

    For the `--url` flag, use the connection string you got from the CockroachCloud Console [earlier](#get-the-connection-string):
       - Replace `<username>` and `<password>` with the SQL user and password that you created.
       - Replace `<certs_dir>/<ca.crt>` with the path to the CA certificate that you downloaded.

1. In the SQL shell, create the `bank` database that your application will use:

    {% include copy-clipboard.html %}
    ~~~ sql
    > CREATE DATABASE bank;
    ~~~

1. Give your user the necessary permissions:

    {% include copy-clipboard.html %}
    ~~~ sql
    > GRANT ALL ON DATABASE bank TO <username>;
    ~~~

</section>

## Step 4. Run the Python code

Now that you have a database, you'll run the code shown below to:

- Create an `accounts` table and insert some rows.
- Transfer funds between two accounts inside a [transaction](transactions.html). To ensure that you [handle transaction retry errors](transactions.html#client-side-intervention), you'll use an application-level retry loop that, in case of error, sleeps before trying the funds transfer again. If it encounters another retry error, it sleeps for a longer interval, implementing [exponential backoff](https://en.wikipedia.org/wiki/Exponential_backoff).
- Finally, you'll delete the accounts from the table before exiting so you can re-run the example code.

### Download the code

Download the <a href="https://raw.githubusercontent.com/cockroachdb/docs/master/_includes/{{page.version.version}}/app/basic-sample.py" download><code>basic-sample.py</code></a> file, or create the file yourself and copy the code into it.

{% include copy-clipboard.html %}
~~~ python
{% include {{page.version.version}}/app/basic-sample.py %}
~~~

If you prefer, you can also clone a version of the code:

{% include copy-clipboard.html %}
~~~ shell
$ git clone https://github.com/cockroachlabs/hello-world-python-psycopg2/
~~~

### Update the connection parameters

<section class="filter-content" markdown="1" data-scope="local">

In the `main()` function, update the connection parameters to match the `(sql/tcp)` connection string you got from SQL shell welcome text [earlier](#step-2-start-cockroachdb):

- Change `user` to `'root'`.
- Add `password` parameter and set it to `'admin'`.
- Remove `sslrootcert`, `sslkey` and `sslcert`. You do not need to specify certificates to connect to a secure demo cluster.
- Change `host` and `port` to the hostname and port in connection string from SQL shell welcome text.

</section>

<section class="filter-content" markdown="1" data-scope="cockroachcloud">

In the `main()` function, update the connection parameters to match the connection string you got from the CockroachCloud Console [earlier](#get-the-connection-string):

- Change `user` to the SQL user that you created.
- Add `password` and set it to the SQL user's password that you created.
- Change `sslmode` to `verify-full`.
- Change `sslrootcert` to the path to the CA certificate that you downloaded.
- Remove `sslkey` and `sslcert`. To connect to a CockroachCloud cluster, you need only `sslrootcert` and the user's password.
- Change `host` to the hostname in the connection string from CockroachCloud.

</section>

### Run the code

{% include copy-clipboard.html %}
~~~ shell
$ python basic-sample.py
~~~

The output should show the account balances before and after the funds transfer:

~~~
Balances at Fri Sep 18 00:11:10 2020
['1', '1000']
['2', '250']
Balances at Fri Sep 18 00:11:10 2020
['1', '900']
['2', '350']
~~~

## What's next?

Read more about using the [Python psycopg2 driver](http://initd.org/psycopg/docs/).

{% include {{page.version.version}}/app/see-also-links.md %}
