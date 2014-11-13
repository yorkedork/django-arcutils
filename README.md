# ARC Utils

## Install

    pip install -e git://github.com/PSU-OIT-ARC/django-arcutils.git#egg=django-arcutils

If you are using LDAP, `pip install python3-ldap`

Add to settings file:

    INSTALLED_APPS = (
        'arcutils',
    )

Optionally, add your LDAP connection information

    LDAP = {
        "default": {
            "host": "ldap://ldap-bulk.oit.pdx.edu",
            "username": "rethinkwebsite,ou=service,dc=pdx,dc=edu",
            "password": "foobar",
            "search_dn": "ou=people,dc=pdx,dc=edu",
            "ca_file": "/path/to/ca_file.crt",
        }
    }

If you'd like to specify a custom number of requests that can be made before Django clears the sessions, add the following line as well (replacing 1000 with a positive integer of your choice)

    MAX_REQUESTS_BEFORE_SESSION_CLEAR = 1000
    
## Features

1. Monkey Patch `django.contrib.auth.forms.PasswordResetForm` so that it raises an error if the email address is not found in the DB
1. Adds the `bootstrap` template tag and all arcutils template tags to the builtin template tags
1. `arcutils.will_be_deleted_with(obj)` yields a two tuple -- a model class, and a set of objects -- that would be deleted if obj were deleted. This is useful on your delete views so you can list the objects that will be deleted in a cascading manner.
1. `arcutils.ChoiceEnum`
```python
class FooType(ChoiceEnum):
    A = 1
    B = 2

    _choices = (
        (A, "Alpha"),
        (B, "Beta"),
    )

# in your model somewhere

class SomeModel(models.Model):
    foo = models.ChoiceField(choices=FooType)

```
1. `arcutils.dictfetchall` pass a cursor, and get the rows back as a dict
1. `arcutils.ldap.escape` is an alias for `ldap.filter.escape_filter_chars`
1. `arcutils.ldap.ldapsearch(query, using='default', **kwargs)` performs an LDAP search using the LDAP connection specified by the using parameter.
1. `arcutils.ldap.parse_profile()` will parse out the first_name, last_name, email, and odin as a dict from an ldap result.
```python
results = ldapsearch("odin=mdj2")
dn, entry = results[0]
parse_profile(entry)
```
1. `arcutils.BaseFormSet` and `arcutils.BaseModelFormSet` have an iter_with_empty_form_first() that is is basically `([formset.empty_form] + formset.forms)`. This makes it convenient to iterate over the empty form in templates, without having a special case for it.
1. `arcutils.BaseFormSet` and `arcutils.BaseModelFormSet` override the clean method, so that if a form is being deleted, its validation errors are blanked out.
1. `arcutils.session_monitor` counts the number of requests made to the server, then clears the session data when the count has reached a certain limit (default: 1000).

## Testing

    pip install model_mommy django mock python-ldap
    ./runtests.py
