..  Copyright (c) 2014-present PlatformIO <contact@platformio.org>
    Licensed under the Apache License, Version 2.0 (the "License");
    you may not use this file except in compliance with the License.
    You may obtain a copy of the License at
       http://www.apache.org/licenses/LICENSE-2.0
    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.

.. _projectconf_interpolation:

Interpolation of Values
-----------------------

On top of the core functionality, PlatformIO supports interpolation. It enables values
to contain format strings which refer to other values from foreign sections.

Interpolation has the next syntax – ``${<section>.<option>}``, where
``<section>`` is a value from ``[<section>]`` group, and ``<option>`` is a
first item from pair ``<option> = value``.

.. list-table::
    :header-rows:  1

    * - Syntax
      - Meaning
      - Example

    * - ``${sysenv.<name>}``
      - Embed system environment variable by a name
      - ``${sysenv.HOME}`` refers to user home directory on Unix machine

    * - ``${platformio.<option>}``
      - Embed value from :ref:`projectconf_section_platformio`
      - ``${platformio.packages_dir}`` refers to a path of :ref:`projectconf_pio_packages_dir`

    * - ``${env.<option>}``
      - Embed default value from :ref:`projectconf_section_env`
      - ``${env.debug_build_flags}`` refers to the default debugging flags

    * - ``${<section>.<option>}``
      - Embed value from another section.
      - ``${extra.lib_deps}`` embeds library dependencies declared in the section named ``extra``

    * - ``${this.<option>}``
      - Embed value from the current section without declaring a section name
      - ``${this.board}`` refers to the "board" value of the current section

    * - ``${this.__env__}``
      - Embed environment name of the current section. The section must start with ``env:`` prefix
      - ``${this.__name__}`` will embed "foo" value being declared in the "env:foo" section

* Interpolation can span multiple levels
* Interpolation can be applied only for the option's value
* Multiple interpolations are allowed
* The :ref:`projectconf_section_platformio` and :ref:`projectconf_section_env`
  sections are reserved and could not be used as a custom section. Some good
  section names might be ``extra`` or ``custom``.

.. note::
    If you need to **share common configuration options** between build
    environments, please take a look at **"Global scope"** in
    :ref:`projectconf_section_env` or :ref:`projectconf_env_extends` option which
    allows extending of other sections.

**Example:**

.. code-block:: ini

    ; COMMON options
    ; Each "[env:***]" extends this "[env]" by default
    [env]
    framework = arduino
    build_flags = -D VERSION=1.2.3

    ; CUSTOM options
    ; You need manually inject these options into a section
    ; using ${extra.<name_of_option>} (see below)
    [extra]
    lib_deps_builtin =
      SPI
      Wire
    lib_deps_external =
      bblanchon/ArduinoJson@>5.6.0

    [env:uno]
    platform = atmelavr
    board = uno
    lib_deps =
      ${extra.lib_deps_builtin}
      ${extra.lib_deps_external}
    platform_packages =
      platformio/tool-simavr
    test_speed = 9600
    test_testing_command =
      ${platformio.packages_dir}/tool-simavr/bin/simavr
      -m
      atmega328p
      -f
      16000000L
      ${platformio.build_dir}/${this.__env__}/firmware.elf

    [env:esp32dev]
    platform = espressif32
    board = esp32dev
    build_flags = ${env.build_flags} -Wall
    lib_deps =
      ${extra.lib_deps_builtin}
      ${extra.lib_deps_external}
      knolleary/PubSubClient @ ~2.6
      paulstoffregen/OneWire @ ^2.3.5

    ; Keep sensitive data in environment variables
    ;
    ; Unix
    ; export WIFI_SSID='\"my\ ssid\ name\"'
    ; export WIFI_PASS='\"my\ password\"'
    ;
    ; Windows
    ; set WIFI_SSID='"my ssid name"'
    ; set WIFI_PASS='"my password"'


.. warning::

    Be careful with special characters in system environment variables on Unix systems,
    especially when they are used as the value for preprocessor directives.
    Symbols like ``$``, ``&``, ``~``, etc must be explicitly escaped, for example:

    .. code-block:: bash

      export WIFI_PASS='\"my\~p\&a\\\$\$\$\$word\"'
