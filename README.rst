.. image:: https://img.shields.io/badge/docs-passing-green.svg
   :target: https://nornir.tech/nornir_jinja2/
   :alt: Documentation

.. image:: https://github.com/nornir-automation/nornir_jinja2/workflows/test_nornir_jinja2/badge.svg
   :target: https://github.com/nornir-automation/nornir_jinja2/actions?query=workflow%3Atest_nornir_jinja2
   :alt: test_nornir_jinja2

nornir_jinja2
=============

Collection of simple plugins for `nornir <github.com/nornir-automation/nornir/>`_

Installation
------------

.. code::

    pip install nornir-jinja2

Plugins
-------

Tasks
_____

* **template_file** - Render a jinja2 template from a file
* **template_string** Render a jinja2 template from a string

Usage
-----

Arguments:
    - template (str): The filename of the Jinja2 template.
    - path (str): The path to the directory where Jinja2 template is located.
    - jinja_filters (dict, optional): A dictionary of custom Jinja2 filters for data manipulation within the template.
    - jinja_env (jinja2.Environment, optional): A custom Jinja2 environment object for more fine-grained template control.
    - formatFunc (callable, optional): A formatting function to post-process the rendered results.

Detailed Parameter Explanations
------------------------------

template (str)

*   Purpose: Specifies the Jinja2 template file that defines the structure and content of the rendered result.

path (str)

*   Purpose: Specifies the directory where the Jinja2 template file is located.

jinja_filters (dict, optional)

*   Purpose: Provides custom Jinja2 filters for transforming or formatting data within the template.

jinja_env (jinja2.Environment, optional)

*   Purpose: Provides a custom Jinja2 environment for configuring template loaders, syntax options, etc.

formatFunc (callable, optional)

*   Purpose: Post-processes the rendered results, e.g., formatting them as XML, JSON, or other formats to make develop easier.


Example
-------

In Nornir main script, use below code for example:

.. code:: python

    result = task.run(
        task="NXOS VLAN",
        template="vlan.j2",
        path="./templates/NXOS/features",
        jinja_filters={"rep": jinja_replacement},
        formatFunc=format_xml
    )

In the Jinja2 template, you can use the filter as below:

.. code:: text

        {% set vlan = host.facts.VLAN %}
        {% for key, value in vlan.items() %}
            {% if key == "vlan_id" %}
                <{{ key | rep }}>vlan-{{ value }}</{{ key | rep }}>
            {% else %}
                <{{ key | rep }}>{{ value }}</{{ key | rep }}>
            {% endif %}
        {% endfor %}

To use replacement filter, you can define a function as below in the Nornir main script or somewhere else:

.. code:: python

     def jinja_replacement(s):
        f = open("./script/NXOS/replacement.json")
        rep = json.load(f)
        f.close()

        for key, value in rep.items():
            s = s.replace(key, value)
        return s

In `replacement.json` file, you can define the key value pairs as below:

.. code:: json

    {
        "description": "descr",
        "vlan_id": "fabEncap",
        "interface-vlan": "ifvlan"
    }

To use format function, you can define a function as below in the Nornir main script:

.. code:: python

    def format_json(text: str) -> str:
        try:
            json.loads(text)
        except ValueError as e:
            return text

        reform = json.dumps(json.loads(text), indent=4)

        return reform


    def format_xml(text: str) -> str:
        try:
            elementTree.fromstring(text)
        except elementTree.ParseError as e:
            return text

        reform = minidom.parseString(text).toprettyxml(indent="      ")
        reform = re.sub(r'^<\?xml.*?\?>', '', reform).strip()
        reform = re.sub(r'\n\s*\n', '\n', reform)
        reform = re.sub(r'&quot;', '"', reform)

        return reform