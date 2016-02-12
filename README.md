## This version is specific for cats4gold. DO NOT MERGE into the forked version.
# Marketing API SDK for Python

[![Build Status](https://travis-ci.org/facebook/facebook-python-ads-sdk.svg)](https://travis-ci.org/facebook/facebook-python-ads-sdk)

## The Ads SDK for Python provides an easy interface and abstraction to the Marketing API.

Python is currently the most popular language for our third party ads
developers. ``facebookads`` is a Python package that provides an interface
between your Python application and Facebook's
<a href="https://developers.facebook.com/docs/marketing-apis">Marketing API</a>. This
tutorial covers the basics knowledge needed to use the SDK and provide some
exercises for the reader.

**NOTE**: ``facebookads`` package is compatible with Python 2 and 3!

## Pre-requisites

### An App

To get started with the SDK you must have a Facebook app
<a href="https://developers.facebook.com/">registered on
developers.facebook.com</a>.

**IMPORTANT**: Enable all migrations in the App's Settings->Migrations page.

**IMPORTANT**: For extra security, the SDK requires that you turn on 'App Secret
Proof for Server API calls' in your app's Settings->Advanced page.

Your app should now be able to use the Marketing API!

### An Access Token

You need to generate a user access token for your app and ask for the
``ads_management`` permission. It is expected that an app in production will
build its own infrastructure to interact with a user to generate an access token
and choose an account to manage.
<a href="https://developers.facebook.com/docs/marketing-api/using-the-api">Learn
more about access tokens here</a>.

For now, we can use the
<a href="https://developers.facebook.com/tools/explorer">Graph Explorer</a> to
get an access token.

## Install package

The easiest way to install the SDK is via ``pip`` in your shell.

**NOTE**: For Python 3, use ``pip3`` and ``python3`` instead.

**NOTE**: Use ``sudo`` if any of these complain about permissions. (This might
happen if you are using a system installed Python.)

If you don't have pip:

```
easy_install pip
```

Now execute when you have pip:

```
pip install facebookads
```

If you care for the latest version instead of a possibly outdated version in the
<a href="https://pypi.python.org">pypi.python.org</a> repository,
<a href="https://github.com/facebook/facebook-python-ads-sdk">check out the
repository from GitHub or download a release tarball</a>. Once you've got the
package downloaded and unzipped, install it:

```
python setup.py install
```

Great, now you are ready to use the SDK!

## Bootstrapping

The rest of the example code given will assume you have bootstrapped the api
into your program like the following sample app:

```python
from facebookads.api import FacebookAdsApi
from facebookads import objects

my_app_id = '<APP_ID>'
my_app_secret = '<APP_SECRET>'
my_access_token = '<ACCESS_TOKEN>'
FacebookAdsApi.init(my_app_id, my_app_secret, my_access_token)
```

**NOTE**: We shall use the objects module throughout the rest of the tutorial.

## Understanding CRUD

The SDK implements a CRUD (create, read, update, delete) design. Objects
relevant to exploring the graph are located in the objects module of the
facebookads package.

All objects on the graph are instances of ``AbstractObject``. Some objects can
be directly queried and thus are instances of ``AbstractCrudObject`` (a subclass
of ``AbstractObject``). Both these abstract classes are located in
``facebookads.objects``.

AbstractCrudObject can have all or some of the following methods:

* ``remote_create``
* ``remote_read``
* ``remote_update``
* ``remote_delete``

For example, Campaign has all these methods but AdAccount does not. Read the
Marketing API documentation for more information about
<a href="https://developers.facebook.com/docs/marketing-api/reference">how different ad
objects are used</a>.

## Exploring the Graph

The way the SDK abstracts the API is by defining classes that represent objects
on the graph. These class definitions and their helpers are located in
``facebookads.objects``.

### Initializing Objects

Look at ``AbstractObject``'s and ``AbstractCrudObject``'s ``__init__`` method
for more information. Most objects on the graph subclass from one of the two.

When instantiating an ad object, you can specify its id if it already exists by
defining ``fbid`` argument. You can specify an object's parent id as well by
defining the ``parent_id`` argument. Lastly, if you want to interact with the
API using a specific api object instead of the default, you can specify the
``api`` argument.

### Edges

Look at the methods of an object to see what associations over which we can
iterate. For example an ``AdUser`` object has a method ``get_ad_accounts`` which
returns an iterator of ``AdAccount`` objects.

### Ad Account

Most ad-related operations are in the context of an ad account. You can go to
<a href="https://www.facebook.com/ads/manage">Ads Manager</a> to see accounts
for which you have permission. Most of you probably have a personal account.

Let's get all the ad accounts for the user with the given access token. I only
have one account so the following is printed:

```python
>>> me = objects.AdUser(fbid='me')
>>> my_accounts = list(me.get_ad_accounts())
>>> print(my_accounts)
[{   'account_id': u'17842443', 'id': u'act_17842443'}]
>>> type(my_accounts[0])
<class 'facebookads.objects.AdAccount'>
```

**WARNING**: We do not specify a keyword argument ``api=api`` when instantiating
the ``AdUser`` object here because we've already set the default api when
bootstrapping.

**NOTE**: We wrap the return value of ``get_ad_accounts`` with ``list()``
because ``get_ad_accounts`` returns an ``EdgeIterator`` object (located in
``facebookads.objects``) and we want to get the full list right away instead of
having the iterator lazily loading accounts.

For our purposes, we can just pick an account and do our experiments in its
context:

```python
>>> my_account = my_accounts[0]
```

Or if you already know your account id:

```python
>>> my_account = objects.AdAccount('act_17842443')
```

## Create

Let's create a campaign. It's in the context of the account, i.e. its parent
should be the account.

```python
campaign = objects.Campaign(parent_id = my_account.get_id_assured())
```

Then we specify some details about the campaign. To figure out what properties
to define, you should look at the available fields of the object (located in
``Campaign.Field``) and also look at the ad object's documentation (e.g.
<a href="https://developers.facebook.com/docs/marketing-api/reference/ad-campaign-group">
Campaign</a>).

**NOTE**: To find out the fields, use Python's builtin help function:
``help(objects.Campaign.Field)`` or look at ``facebookads/objects.py``.

```python
campaign[objects.Campaign.Field.name] = "Potato Campain" # sic
campaign[objects.Campaign.Field.configured_status] = objects.Campaign.Status.paused
```

Finally, we make the create request by calling the ``remote_create`` method.

```python
campaign.remote_create()
```

If there's an error, an exception will be raised. Possible exceptions and their
descriptions are listed in ``facebookads.exceptions``.

## Read

We can also read properties of an object from the api assuming that the object
is already created and has a node path. Accessing properties of an object is
simple since ``AbstractObject`` implements the ``collections.MutableMapping``.
You can access them just like accessing a key of a dictionary:

```python
>>> print(my_account)
{'account_id': u'17842443', 'id': u'act_17842443'}
>>> my_account.remote_read(fields=[objects.AdAccount.Field.amount_spent])
>>> print(my_account[objects.AdAccount.Field.amount_spent])
{'amount_spent': 21167, 'account_id': u'17842443', 'id': u'act_17842443'}
```

## Update

To update an object, we can modify its properties and then call the
``remote_update`` method to sync the object with the server. Let's correct the
typo "Campain" to "Campaign":

```python
>>> campaign[objects.Campaign.Field.name] = "Potato Campaign"
>>> campaign.remote_update()
```

You can see the results in ads manager.

## Delete

If we decide we don't want the campaign we created anymore:

```python
campaign.remote_delete()
```

## Useful Arguments

### MULTIPLE ACCESS TOKENS

Throughout the docs, the method FacebookAdsApi.init is called before making any API calls. This
method set up a default FacebookAdsApi object to be used everywhere. That simplifies the usage
but it's not feasible when a system using the SDK will make calls on behalf of multiple users.

The reason why this is not feasible is because each user should have its own FacebookSession, with its own
access token, rather than using the same session for every one. Each session should be used to create a
separate FacebookAdsApi object. See example below:


```python
my_app_id = '<APP_ID>'
my_app_secret = '<APP_SECRET>'
my_access_token_1 = '<ACCESS_TOKEN_1>'
my_access_token_2 = '<ACCESS_TOKEN_2>'

session1 = FacebookSession(
    my_app_id,
    my_app_secret,
    my_access_token_1,
)

session2 = FacebookSession(
    my_app_id,
    my_app_secret,
    my_access_token_2,
)

api1 = FacebookAdsApi(session1)
api2 = FacebookAdsApi(session2)
```
In the SDK examples, we always set a single FacebookAdsApi object as the default one.
However, working with multiples access_tokens, require us to use multiples apis. We may set a default
api for a user, but, for the other users,  we shall use its the api object as a param. In the example below,
we create two AdUsers, the first one using the default api and the second one using its api object:

```python
FacebookAdsApi.set_default_api(api1)

me1 = AdUser(fbid='me')
me2 = AdUser(fbid='me', api=api2)
```
Another way to create the same objects from above would be:

```python
me1 = AdUser(fbid='me', api=api1)
me2 = AdUser(fbid='me', api=api2)
```
From here, all the following workflow for these objects remains the same. The only exceptions are
the classmethods calls, where we now should pass the api we want to use as the last parameter
on every call. For instance, a call to the Aduser.get_by_ids method should be like this:

```python
session = FacebookSession(
 my_app_id,
 my_app_secret,
 my_access_token_1,
)

api = FacebookAdsApi(session1)
Aduser.get_by_ids(ids=['<UID_1>', '<UID_2>'], api=api)
```
### CRUD

All CRUD calls support a ``params`` keyword argument which takes a dictionary
mapping parameter names to values in case advanced modification is required. You
can find the list of parameter names as attributes of
``{your object class}.Field``. Under the Field class there may be other classes
which contain, as attributes, valid fields of the value of one of the parent
properties.

``remote_create`` and ``remote_update`` support a ``files`` keyword argument
which takes a dictionary mapping file reference names to binary opened file
objects.

``remote_read`` supports a ``fields`` keyword argument which is a convenient way
of specifying the 'fields' parameter. ``fields`` takes a list of fields which
should be read during the call. The valid fields can be found as attributes of
the class Field.

### Edges

When initializing an ``EdgeIterator`` or when calling a method such as
``AdAccount.get_ad_campaigns``:

* You can specify a ``fields`` argument which takes a list of fields to read for
the objects being read.
* You can specify a ``params`` argument that can help you specify or filter the
edge more precisely.

## Batch Calling

It is efficient to group together large numbers of calls into one http request.
The SDK makes this process simple. You can group together calls into an instance
of ``FacebookAdsApiBatch`` (available in facebookads.api). To easily get one
for your api instance:

```python
my_api_batch = api.new_batch()
```

Calls can be added to the batch instead of being executed immediately:

```python
campaign.remote_delete(batch=my_api_batch)
```

Once you're finished adding calls to the batch, you can send off the request:

```python
my_api_batch.execute()
```

Please follow <a href="https://developers.facebook.com/docs/graph-api/making-multiple-requests">
batch call guidelines in the Marketing API documentation</a>. There are optimal
numbers of calls per batch. In addition, you may need to watch out that for rate
limiting as a batch call simply improves network performance and each call does
count individually towards rate limiting.

## Exceptions

See ``facebookads.exceptions`` for a list of exceptions which may be thrown by
the SDK.

## Tests

### Unit tests

The unit tests don't require an access token or network access. Run them
with your default installed Python as follows:

```
python -m facebookads.test.unit
```

You can also use tox to run the unit tests with multiple Python versions:

```
sudo apt-get install python-tox  # Debian/Ubuntu
sudo yum install python-tox      # Fedora
tox --skip-missing-interpreters
```

You can increase interpreter coverage by installing additional versions of
Python. On Ubuntu you can use the
[deadsnakes PPA](https://launchpad.net/~fkrull/+archive/ubuntu/deadsnakes).
On other distributions you can
[build from source](https://www.python.org/downloads/) and then use
`sudo make altinstall` to avoid conflicts with your system-installed
version.

### Integration tests

The integration tests require an access token with ads_management scope.
You can obtain a short-lived token from the
[Graph API Explorer](https://developers.facebook.com/tools/explorer/).
These tests access the live Facebook API but shouldn't actually
launch an ad or spend any money.

Copy the `config.json.example` to `config.json` and fill in the appropriate
details.

```
python -m facebookads.test.integration <ACCESS_TOKEN>
# Access token not required if it's defined in config.json
```

## Examples

Examples of usage are located in the ``examples/`` folder.
