====================================
Django REST Framework JSON CamelCase
====================================

.. image:: https://travis-ci.org/vbabiy/djangorestframework-camel-case.svg?branch=master
        :target: https://travis-ci.org/vbabiy/djangorestframework-camel-case

.. image:: https://badge.fury.io/py/djangorestframework-camel-case.svg
    :target: https://badge.fury.io/py/djangorestframework-camel-case

Camel case JSON support for Django REST framework.  This affects input and output by default.

============
Installation
============

At the command line::

    $ pip install djangorestframework-camel-case

Add the render and parser to your django settings file as needed.  If you only want responses to be converted to camelCase, you only need the renderer classes, not the parsers (but check the settings).

.. code-block:: python

    # ...
    REST_FRAMEWORK = {

        'DEFAULT_RENDERER_CLASSES': (
            'djangorestframework_camel_case.render.CamelCaseJSONRenderer',
            'djangorestframework_camel_case.render.CamelCaseBrowsableAPIRenderer',
            # Any other renders
        ),

        'DEFAULT_PARSER_CLASSES': (
            # If you use MultiPartFormParser or FormParser, we also have a camel case version
            'djangorestframework_camel_case.parser.CamelCaseFormParser',
            'djangorestframework_camel_case.parser.CamelCaseMultiPartParser',
            'djangorestframework_camel_case.parser.CamelCaseJSONParser',
            # Any other parsers
        ),
    }
    # ...

Add query param middleware to django settings file.

.. code-block:: python

    # ...
    MIDDLEWARE = [
        # Any other middleware
        'djangorestframework_camel_case.middleware.CamelCaseMiddleWare',
    ]
    # ...

=================
Swapping Renderer
=================

By default the package uses `rest_framework.renderers.JSONRenderer`. If you want
to use another renderer, the two possible are:

`drf_orjson_renderer.renderers.ORJSONRenderer` or
`drf_ujson.renderers.UJSONRenderer` or
`rest_framework.renderers.UnicodeJSONRenderer` for DRF < 3.0, specify it in your django
settings file.

.. code-block:: python

    # ...
    JSON_CAMEL_CASE = {
        'RENDERER_CLASS': 'drf_orjson_renderer.renderers.ORJSONRenderer'
    }
    # ...

=====================
Underscoreize Options
=====================

Normalize Inputs
----------------

By default, the middleware normalizes any incoming query parameters and other inputs from 
camelCase to snake_case so everything passes through the same logic.  If you do not want this,
disable the `normalize_inputs` setting:

.. code-block:: python

    REST_FRAMEWORK = {
        # ...
        "JSON_UNDERSCOREIZE": {
            # ...
            "normalize_inputs": False,
            # ...
        },
        # ...
    }


No Underscore Before Number
---------------------------


As raised in `this comment <https://github.com/krasa/StringManipulation/issues/8#issuecomment-121203018>`_
there are two conventions of snake case.

.. code-block:: text

    # Case 1 (Package default)
    v2Counter -> v_2_counter
    fooBar2 -> foo_bar_2

    # Case 2
    v2Counter -> v2_counter
    fooBar2 -> foo_bar2


By default, the package uses the first case. To use the second case, specify it in your django settings file.

.. code-block:: python

    REST_FRAMEWORK = {
        # ...
        'JSON_UNDERSCOREIZE': {
            'no_underscore_before_number': True,
        },
        # ...
    }

Alternatively, you can change this behavior on a class level by setting `json_underscoreize`:

.. code-block:: python

    from djangorestframework_camel_case.parser import CamelCaseJSONParser
    from rest_framework.generics import CreateAPIView

    class NoUnderscoreBeforeNumberCamelCaseJSONParser(CamelCaseJSONParser):
        json_underscoreize = {'no_underscore_before_number': True}

    class MyView(CreateAPIView):
        queryset = MyModel.objects.all()
        serializer_class = MySerializer
        parser_classes = (NoUnderscoreBeforeNumberCamelCaseJSONParser,)


Ignore Fields
-------------


You can also specify fields which should not have their data changed.
The specified field(s) would still have their name change, but there would be no recursion.
For example:

.. code-block:: python

    data = {"my_key": {"do_not_change": 1}}

Would become:

.. code-block:: python

    {"myKey": {"doNotChange": 1}}

However, if you set in your settings:

.. code-block:: python

    REST_FRAMEWORK = {
        # ...
        "JSON_UNDERSCOREIZE": {
            # ...
            "ignore_fields": ("my_key",),
            # ...
        },
        # ...
    }

The `my_key` field would not have its data changed:

.. code-block:: python

    {"myKey": {"do_not_change": 1}}
    

Ignore Keys
-----------


You can also specify keys which should *not* be renamed.
The specified field(s) would still change (even recursively).
For example:

.. code-block:: python

    data = {"unchanging_key": {"change_me": 1}}

Would become:

.. code-block:: python

    {"unchangingKey": {"changeMe": 1}}

However, if you set in your settings:

.. code-block:: python

    REST_FRAMEWORK = {
        # ...
        "JSON_UNDERSCOREIZE": {
            # ...
            "ignore_keys": ("unchanging_key",),
            # ...
        },
        # ...
    }

The `unchanging_key` field would not be renamed:

.. code-block:: python

    {"unchanging_key": {"changeMe": 1}}

ignore_keys and ignore_fields can be applied to the same key if required.


Preserve Underscore Keys
------------------------


If you need to preserve the underscore keys alongside the camel case versions for compatibility or other reasons, specify that option:

.. code-block:: python

    REST_FRAMEWORK = {
        # ...
        "JSON_UNDERSCOREIZE": {
            # ...
            "preserve_underscore_keys": True,
            # ...
        },
        # ...
    }
    
For example:

.. code-block:: python

    data = {"original_key": {"another_original_key": 1}}

Would become:

.. code-block:: python

    {
        "originalKey": {
            "anotherOriginalKey": 1
        },
        "original_key": {
            "another_original_key": 1
        }
    }


Ignore Request Paths
--------------------

Entire requests can be ignored by the JSON renderer.

.. code-block:: python

    REST_FRAMEWORK = {
        # ...
        "JSON_UNDERSCOREIZE": {
            # ...
            "ignore_paths": [
                '/api/v1/my_custom_endpoint/'
            ],
            # ...
        },
        # ...
    }
    
With this option set, `/api/v1/my_custom_endpoint/` would not pass through the custom renderer.


Custom Keys
--------------------

If there are keys for which the normal underscore-to-camel case conversion is not appropriate, you can specify custom keys.

.. code-block:: python

    REST_FRAMEWORK = {
        # ...
        "JSON_UNDERSCOREIZE": {
            # ...
            "custom_key_map": {
                'my_custom_id': 'myCustomID',
                'my_custom_uuid': 'myCustomUUID',
            },
            # ...
        },
        # ...
    }

With this option set, the keys would be transformed as follows:

- `my_custom_id`: `myCustomID` instead of the default `myCustomId`
- `my_custom_uuid`: `myCustomUUID` instead of the default `myCustomUuid`


=============
Running Tests
=============

To run the current test suite, execute the following from the root of he project::

    $ python -m unittest discover


=======
License
=======

* Free software: BSD license
