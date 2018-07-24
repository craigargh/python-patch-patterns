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

Code to test (`orders.py`):

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

### pymysql fetchall()

Code to test (`orders.py`):

```python
import pymysql

def query_items():
    connection = pymysql.connect(
        db='users',
        host='127.01.54.12',
        user='admin',
        passwd='gottagofast',
        charset='utf8',
    )

    sql = 'SELECT * FROM orders.items'

    with connection as cursor:
        cursor.execute(sql)

        items = cursor.fetchall()

    return items
```

Test:

```python
from unittest import TestCase
form unitests.mock import patch

from orders import query_items


class TestOrders(TestCase):
    
    @patch('orders.pymysql')
    def test_check_started_returns_result_of_query(self):
        item_records_stub = [
            (12, 'a box'),
            (13, 'jumper'),
            (14, 'shoes'),
        ]

        self.mysql_mock \
            .connect.return_value \
            .__enter__.return_value \
            .fetchall.return_value = item_records_stub

        result = query_items()

        self.assertEqual(3, len(result))
        
        self.assertEqual((12, 'a box'), result[0])
        self.assertEqual((13, 'jumper'), result[1])
        self.assertEqual((14, 'shoes'), result[2])
```


### pymysql execute_many()


## datetime

### datetime.datetime.today()


Code to test (`timings.py`):

```python
import datetime


def current_date():
    today = datetime.datetime.today()

    return today.strftime('%Y-%m-%d')
```

Test:

```python
import datetime
from unittest import TestCase
from unitests.mock import patch

from timings import current_date


class TestTimings(TestCase):

    def setUp():
        self.datetime_mock = patch('timings.datetime.datetime',
                                   Mock(wraps=datetime.datetime)).start()
        
        self.datetime_mock.today.return_value = datetime.datetime(1924, 6, 16, 6, 32, 8)

        self.addCleanup(patch.stopall)

    def test_current_date_is_returned_as_a_string():
        result = current_date()

        self.assertEqual('1924-06-16', result)
```

Alternatively, use the `freezegun` package.
