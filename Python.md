# Python programming language

The general suggestion of writing tests applies here. Try to test all of the
code that is being written and don't delay that for later (it will not get
easier).

## Code style

Note: While this is section describes an ideal style, we know that parts of MetaBrainz
projects do not follow these guidelines exactly. We are pragmatic in our application of the
guidelines and try our best to follow them, but there will be deviations. When in doubt, follow
the guidelines rather than copying existing code.

As most of the other projects written in Python, we use [PEP 8]
(https://www.python.org/dev/peps/pep-0008/).  We change some of the recomendations:

* Maximum line length (79 characters). The general limit we have is somewhere around 120-130.

*Reccomended video: "[Beyond PEP 8 -- Best practices for beautiful intelligible code]
(https://www.youtube.com/watch?v=wf-BqAjZb8M)" by Raymond Hettinger at PyCon 2015,
which talks about the famous P versus NP problem.*

If the project has a `pylintrc` file, you can use pylint to check for potential style violations:

    pylint --rcfile=pylintrc

The general idea is to make the code within a project consistent and easy to interpret (for humans).


### Docstrings

Include docstrings in functions. This should say *what* the function does, its *inputs*,
and *outputs* or possible *failure states* (exceptions).

Unless otherwise stated, we use "Google-style" docstrings:
https://google.github.io/styleguide/pyguide.html?showone=Comments#Comments.

For flask application docstrings for views are turned into API Documentation
(See an example for AcousticBrainz at http://acousticbrainz.readthedocs.io/)

View docstrings should be formatted to include query format and an example of the response:

```python
"""Get low-level data for many recordings at once.

**Example response**:

.. sourcecode:: json

   {"mbid1": {"some": "json"}}

:query recording_ids: *Required.* A list of recording MBIDs to retrieve

  Takes the form `mbid[:offset];mbid[:offset]`. Offsets are optional, and should
  be >= 0
:resheader Content-Type: *application/json*
"""
```

An example of a docstring for a regular method
```python
def frob(foo, bar=None):
    """Frob a foo, with an optional bar operation
       Args:
           foo: the foo to be frobbed
           bar: if set, a callback to a bar operation to be applied after
                the foo is frobbed
       Returns:
           The new foo, after frobbing and baring
```


### SQL

For projects which use sqlalchemy-core, we use `sqlalchemy.text` to write prepared statements. SQL keywords
should be in upper-case and right aligned. For example:

```python
query = sqlalchemy.text("""
    SELECT id,
           display_name,
           email,
           created,
           musicbrainz_id,
           show_gravatar
      FROM "user"
     WHERE musicbrainz_id IN :musicbrainz_usernames
""")
result = connection.execute(query, {'musicbrainz_usernames': tuple(usernames)})
```

It is preferred to assign the query to a separate variable and not include it inside the
`execute` call.

TODO: Projects which don't use sqlalchemy

### Tests

You should write tests for all new methods that you write. If you update a method, make sure that
tests for that method still pass, and write/remove tests as necessary for the change in functionality.

Tests for a module (`package/module.py`) go in a `test` subdirectory - `package/test/test_module.py`

Database tests (for modules in `db`) can use the database. Inherit from `db.testing.DatabaseTestCase`
to automatically reset the database at the end of each testcase.

Try and isolate tests as much as possible. If you have a utility function which transforms or validates
and input, you can test this in isolation.

Webserver tests don't need to write data to the database. Instead, you can use a mock to verify
that the call to the database was made:

```python
@mock.patch("db.data.load_high_level")
def test_hl_numerical_offset(self, hl):
    hl.return_value = {}
    resp = self.client.get("/api/v1/%s/high-level?n=3" % self.uuid)
    self.assertEqual(200, resp.status_code)
    hl.assert_called_with(self.uuid, 3)
```
