.. _django_client:

Django OAuth Client
===================

.. meta::
    :description: The built-in Django integrations for OAuth 1.0 and
        OAuth 2.0 clients, powered by Authlib.

.. module:: authlib.integrations.django_client
    :noindex:

Looking for OAuth providers?

- :ref:`django_oauth1_server`
- :ref:`django_oauth2_server`

The Django client can handle OAuth 1 and OAuth 2 services. Authlib has
a shared API design among framework integrations. Get started with
:ref:`frameworks_clients`.

Create a registry with :class:`OAuth` object::

    from authlib.integrations.django_client import OAuth

    oauth = OAuth()

The common use case for OAuth is authentication, e.g. let your users log in
with Twitter, GitHub, Google etc.

.. note::

    Please read :ref:`frameworks_clients` at first. Authlib has a shared API
    design among framework integrations, learn them from :ref:`frameworks_clients`.

.. versionchanged:: v0.13

    Authlib moved all integrations into ``authlib.integrations`` module since v0.13.
    For earlier version, developers can import the Django client with::

        from authlib.django.client import OAuth


Configuration
-------------

Authlib Django OAuth registry can load the configuration from your Django
application settings automatically. Every key value pair can be omit.
They can be configured from your Django settings::

    AUTHLIB_OAUTH_CLIENTS = {
        'twitter': {
            'client_id': 'Twitter Consumer Key',
            'client_secret': 'Twitter Consumer Secret',
            'request_token_url': 'https://api.twitter.com/oauth/request_token',
            'request_token_params': None,
            'access_token_url': 'https://api.twitter.com/oauth/access_token',
            'access_token_params': None,
            'refresh_token_url': None,
            'authorize_url': 'https://api.twitter.com/oauth/authenticate',
            'api_base_url': 'https://api.twitter.com/1.1/',
            'client_kwargs': None
        }
    }

We suggest that you keep ONLY ``client_id`` and ``client_secret`` in
your application settings, other parameters are better in ``.register()``.

Saving Temporary Credential
---------------------------

In OAuth 1.0, we need to use a temporary credential to exchange access token,
this temporary credential was created before redirecting to the provider (Twitter),
we need to save this temporary credential somewhere in order to use it later.

In OAuth 1, Django client will save the request token in sessions. In this
case, you just need to configure Session Middleware in Django::

    MIDDLEWARE = [
        'django.contrib.sessions.middleware.SessionMiddleware'
    ]

Follow the official Django documentation to set a proper session. Either a
database backend or a cache backend would work well.

.. warning::

    Be aware, using secure cookie as session backend will expose your request
    token.

Routes for Authorization
------------------------

Just like the example in :ref:`frameworks_clients`, everything is the same.
But there is a hint to create ``redirect_uri`` with ``request`` in Django::

    def login(request):
        # build a full authorize callback uri
        redirect_uri = request.build_absolute_uri('/authorize')
        return oauth.twitter.authorize_redirect(request, redirect_uri)


Auto Update Token via Signal
----------------------------

Instead of define a ``update_token`` method and passing it into OAuth registry,
it is also possible to use signal to listen for token updating::

    from django.dispatch import receiver
    from authlib.integrations.django_client import token_update

    @receiver(token_update)
    def on_token_update(sender, name, token, refresh_token=None, access_token=None, **kwargs):
        if refresh_token:
            item = OAuth2Token.find(name=name, refresh_token=refresh_token)
        elif access_token:
            item = OAuth2Token.find(name=name, access_token=access_token)
        else:
            return

        # update old token
        item.access_token = token['access_token']
        item.refresh_token = token.get('refresh_token')
        item.expires_at = token['expires_at']
        item.save()


Django OpenID Connect Client
----------------------------

An OpenID Connect client is no different than a normal OAuth 2.0 client. When
register with ``openid`` scope, the built-in Django OAuth client will handle
everything automatically::

    oauth.register(
        'google',
        ...
        server_metadata_url='https://accounts.google.com/.well-known/openid-configuration',
        client_kwargs={'scope': 'openid profile email'}
    )

When we get the returned token::

    token = oauth.google.authorize_access_token(request)

We can get the user information from the ``id_token`` in the returned token::

    userinfo = oauth.google.parse_id_token(request, token)

Find Django Google login example at https://github.com/authlib/demo-oauth-client/tree/master/django-google-login
