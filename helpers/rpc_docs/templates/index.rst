.. Copyright (c) 2018-2019 The Unit-e developers
   Distributed under the MIT software license, see the accompanying
   file LICENSE or https://opensource.org/licenses/MIT.

.. _rpc:

RPC API Reference
=================

This is the reference for the RPC API calls of unit-e. Use ``unit-e-cli`` to run
the commands.

{% for group, commands in all_commands.items() %}
{{ group }}
{{ group | titleunderline }}

.. toctree::
  :maxdepth: 1

{% for command in commands %}
  {{ command | splitcommand }}
{% endfor %}

{% endfor %}
