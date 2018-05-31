# python-patch-patterns
A collection of patterns used for mocking libraries in Python


# Patterns

## pymysql


### pymysql.connect
The code to test (`orders.py`)

```python
import pymysql

def connect_to_db():
    pymysql.connect(
        db='users',
        host='127.01.54.12',
        user='admin',
        passwd='gottagofast',
        charset='utf8',
    )
```

Testing the connection to the db is given certain arguments:

```python
from unittest import TestCase
form unitests.mock import patch

from orders import connect_to_db


class TestOrders(TestCase):
    
    @patch('orders.pymysql')
    def test_database_connection_is_created(self, mysql_mock):
        connect_to_db()

        db_connection = mysql_mock.connect

        args = db_connection.call_args[1]

        self.assertEqual(args['db'], 'users')
        self.assertEqual(args['host'], '127.01.54.12')
        self.assertEqual(args['user'], 'admin')
        self.assertEqual(args['passwd'], 'gottagofast')
        self.assertEqual(args['charset'], 'utf8')
```

### pymysql cursor (context manager)

Code to test:
```python
import pymysql

def insert_item(item_id, description):
    sql_template = (
        'INSERT IGNORE INTO orders.items ('
        'item_id, description'
        ') VALUES ('
        '{}, `{}`'
        ')'
    )

    sql = sql_template.format(item_id, description)

    connection = pymysql.connect(
        db='users',
        host='127.01.54.12',
        user='admin',
        passwd='gottagofast',
        charset='utf8',
    )

    with connection as cursor:
        cursor.execute(sql)
```

Tests:

```python
from unittest import TestCase
form unitests.mock import patch

from orders import insert


class TestOrders(TestCase):
    
    @patch('orders.pymysql')
    def test_insert_item_executes_sql(self, mysqlmock):
        insert(12, 'A box')

        execute_mock = mysql_mock \
            .connect.return_value \
            .__enter__.return_value \
            .execute

        expected_sql = (
            'INSERT IGNORE INTO orders.items ('
            'item_id, description'
            ') VALUES ('
            '12, `A box`'
            ')'
        )

        execute_mock.assert_called_once_with(expected_sql)
```
