.. meta::
   :description: Hasura Cloud API limits
   :keywords: hasura, docs, cloud, security, limits

.. _api_limits:

API limits
==========

.. contents:: Table of contents
  :backlinks: none
  :depth: 1
  :local:

Introduction
------------

Limiting the depth and/or rate of API requests can help prevent API performance issues caused by malicious or poorly implemented queries. 

Configuring an API limit
------------------------

**Rate limits**
  Restricts number of GraphQL operations per minute. This uses a sliding window approach. This means whenever Hasura Pro receives a request, it will count the rate of that client starting from the current time to last one minute.

  By default, the rate-limit happens on the ``role_name`` i.e the value provided in ``X-HASURA-ROLE``. But you can also combine additional unique parameters for more granularity.
  
  The Unique Parameters that are provided are:

  1. IP Address
  2. Session Variables

  You can choose any one of the above parameters to rate limit the requests.

  **Note**: If you set an `Unique Parameter` then the combination of both the ``role_name`` and the ``Unique Parameter`` will be used to rate-limit requests.
  
  Example:

  If you rate-limit using the ``role_name`` and set the unique parameter for the rate-limit as ``IP Address``, then rate-limit will be applied depending on both those parameteres. i.e If the requests come from a different role but same IP address will **NOT** be rate limited

**Depth limits**
  Restricts a GraphQL operation based on its depth, preventing deeply nested queries.

**Node limits**
  Restricts a GraphQL operation based on the number of nodes. This helps in limiting the number of different pieces of related data to be fetched.

  A node is defined as a field with a selection set. 
  
  For example, in the below query the number of nodes is 3 and they are ``author``, ``articles`` and ``homepage_entries``.

  .. code-block:: graphql

    {
      author {
        name
        articles {
          id 
          title
        }
      }
      homepage_entries {
        article_id
      }
    }

API limits are defined by **role** (anonymous, user) and can restrict request rate, depth, or both. Unique request parameters can include IP address or session variables (*x-hasura-user-id*, *x-hasura-org-id*, etc.)

Manage API limits
-----------------

API limits can have a *global* or *per role* configuration. If an incoming request does not contain a valid role then the global limit is applied.

.. thumbnail:: /img/graphql/cloud/security/pro-tab-apilimits.png
   :alt: Hasura Cloud Console api limit tab

.. admonition:: Admin & IntrospectionQuery exemptions

  All API limits are **not** applied for the admin role, and depth limits are **NOT** applied to introspection queries

Metadata Specification
----------------------

Hasura provides two metadata API's to manage API limits

Setting API Limits
~~~~~~~~~~~~~~~~~~

.. code-block:: yaml


    type: set_api_limits
    args:
      disabled: # Optional Field (Either True or False, The value is False by default)

      depth_limit: # Optional API Limit
        global: # Mandatory Field
        per_role: # Optional Field
          <role_name>: <limit value> # Eg: user: 5
      
      node_limit: # Optional API Limit
        global: # Mandatory Field
        per_role: # Optional Field
          <role_name>: <limit value> # Eg: user: 5

      rate_limit: # Optional API Limit
        global: # Mandatory Field
          unique_params: # Optional Field, Can either be IP Address or Session variables
          max_reqs_per_min: # Mandatory Field

        per_role: # Optional Field
          <role_name>:
            max_reqs_per_min: # Mandatory Field
            unique_params: # Optional Field

In the above metadata spec:

1. The API Limits are ``Enabled`` by default, i.e the default value of ``disabled`` is ``False``
2. When ``disabled`` is `False` and none of the API Limits are set then no API limits are applied.
3. The ``global`` field in all the API Limits is mandatory, and is used as the default API limit if no ``per_role`` option is set for the user.
4. The ``per_role`` can be used to override the `global` API Limit value
5. For ``rate_limit`` if no `unique_params` are provided then, the requests will be rate-limited on the ``role_name`` i.e the ``X-HASURA-ROLE`` that is used to issue the request

Example Metadata Spec:

.. code-block:: yaml


    type: set_api_limits
    args:
      disabled: false

      depth_limit:
        global: 5
        per_role:
          user: 7
      
      node_limit:
        global: 3
        per_role:
          user: 10
   
      rate_limit:
        global:
          unique_params: IP
          max_reqs_per_min: 10
        per_role:
          anonymous:
            max_reqs_per_min: 10
            unique_params: "ip"
          user:
            unique_params:
            - x-hasura-user-id
            - x-hasura-team-id
            max_reqs_per_min: 20




Remove API Limits
~~~~~~~~~~~~~~~~~~

You can remove **all** the api limits that have been set using `remove_api_limit` API.

.. code-block:: yaml


    type: remove_api_limits
    args: {}
