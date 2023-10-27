============
Requirements
============

The following are needed for this package:

* Python 2.7, 3.6
* Django 1.8 - 2.1

Installation
------------

To install fixture-magic from PyPI:

    pip install django-fixture-magic

To install the development version:

    pip install -e git://github.com/ShubhamS021/django-fixture-magic#egg=fixture-magic

For Python 3 compatibility:

    pip install future

Add fixture-magic to your `INSTALLED_APPS` in `settings.py`:

.. code-block:: python

    INSTALLED_APPS = (
        ...
        'fixture_magic',
        ...
    )

Usage
-----

Four commands are available. The `dump_object` command outputs a JSON representation of a specific object and its dependencies:

    ./manage.py dump_object APP.MODEL PK1 PK2 PK3 ... > my_new_fixture.json

Alternatively:

    ./manage.py dump_object APP.MODEL --query '{"pk__in": [PK1, PK2, PK3]}' > my_new_fixture.json

To dump all objects and dependencies, use an asterisk:

    ./manage.py dump_object APP.MODEL '*' > my_new_fixture.json

Loading `my_new_fixture.json` in tests avoids foreign key errors. By default, related fixtures are included. Use `--no-follow` with `dump_object` if the database is partially set up. Example output:

    ./manage.py dump_object APP.Book

.. code-block:: json

    [
      {
          "model": "APP.Author",
          "fields": {
              "pk": 5,
              "name": "Mark Twain",
          }
      },
      {
          "model": "APP.Book",
          "fields": {
              "pk": 2,
              "title": "Tom Sawyer",
              "author": 5
          }
      }
    ]

Output with `--no-follow`:

    ./manage.py dump_object APP.Book --no-follow

.. code-block:: json

    [
      {
          "model": "APP.Book",
          "fields": {
              "pk": 2,
              "title": "Tom Sawyer",
              "author": 5
          }
      }
    ]

Note: Assumes an Author with ID 5 exists in the target database.

`merge_fixtures` combines multiple JSON fixtures into one, removing duplicates:

    ./manage.py merge_fixtures fixture1.json fixture2.json fixture3.json ... > all_my_fixtures.json

`reorder_fixtures` reorganizes fixtures by specified model order to prevent foreign-key errors:

    ./manage.py reorder_fixtures fixture.json APP1.MODEL1 APP2.MODEL2 ... > ordered_fixture.json

Unspecified models are appended at the end.

`custom_dump` allows for complex dumps through `CUSTOM_DUMPS` in settings:

.. code-block:: python

    CUSTOM_DUMPS = {
        'addon': {
            'primary': 'addons.addon',
            'dependents': [
                'current_version',
                'current_version.files.all.0',
            ],
            'order': ('app1.model1', 'app2.model2',),
            'order_cond': {
                'app1.model1': lambda x: 1 if x.get('fields').get('parent_model1') else 0,
                'app2.model2': lambda x: -1 * x.get('pk'),
            },
        }
    }

Dependents, order, and order conditions are optional attributes that modify the dump process.