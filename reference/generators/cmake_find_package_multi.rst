.. _cmake_find_package_multi_generator_reference:


cmake_find_package_multi
========================

.. warning::

    This is a **deprecated** feature. Please refer to the :ref:`Migration Guidelines<conan2_migration_guide>`
    to find the feature that replaced this one.

    The new, **under development** integration with CMake can be found in :ref:`conan_tools_cmake`. This is the integration that will
    become the standard one in Conan 2.0, and the below generators and integrations will be deprecated and removed. While they are
    recommended and usable and we will try not to break them in future releases, some breaking
    changes might still happen if necessary to prepare for the *Conan 2.0 release*.

.. container:: out_reference_box

    This is the reference page for ``cmake_find_package_multi`` generator.
    Go to :ref:`Integrations/CMake<cmake>` if you want to learn how to integrate your project or recipes with CMake.

Generated files
---------------

For each conan package in your graph, it will generate 2 files and 1 more per different ``build_type``.
Being ``<PKG-NAME>`` the package name used in the reference (by default) or the one declared in
``cpp_info.names["cmake_find_package_multi"]`` if specified:

+------------------------------------------+--------------------------------------------------------------------------------------+
| NAME                                     | CONTENTS                                                                             |
+==========================================+======================================================================================+
| | <PKG-NAME>Config.cmake /               | It includes the <PKG-NAME>Targets.cmake and call find_dependency for each dep        |
| | <PKG-NAME>-config.cmake                |                                                                                      |
+------------------------------------------+--------------------------------------------------------------------------------------+
| | <PKG-NAME>ConfigVersion.cmake /        | Package version file for each dep                                                    |
| | <PKG-NAME>-config-version.cmake        |                                                                                      |
+------------------------------------------+--------------------------------------------------------------------------------------+
| | <PKG-NAME>Targets.cmake                | It includes the files *<PKG-NAME>Targets-<BUILD-TYPE>.cmake*                         |
+------------------------------------------+--------------------------------------------------------------------------------------+
| | <PKG-NAME>Targets-debug.cmake          | Specific information for the Debug configuration                                     |
+------------------------------------------+--------------------------------------------------------------------------------------+
| | <PKG-NAME>Targets-release.cmake        | Specific information for the Release configuration                                   |
+------------------------------------------+--------------------------------------------------------------------------------------+
| | <PKG-NAME>Targets-relwithdebinfo.cmake | Specific information for the RelWithDebInfo configuration                            |
+------------------------------------------+--------------------------------------------------------------------------------------+
| | <PKG-NAME>Targets-minsizerel.cmake     | Specific information for the MinSizeRel configuration                                |
+------------------------------------------+--------------------------------------------------------------------------------------+

Targets
-------

A target named ``<PKG-NAME>::<PKG-NAME>`` target is generated with the following properties adjusted:

- ``INTERFACE_INCLUDE_DIRECTORIES``: Containing all the include directories of the package.
- ``INTERFACE_LINK_LIBRARIES``: Library paths to link.
- ``INTERFACE_COMPILE_DEFINITIONS``: Definitions of the library.
- ``INTERFACE_COMPILE_OPTIONS``: Compile options of the library.

The targets contains multi-configuration properties, for example, the compile options property
is declared like this:

.. code-block:: cmake

        set_property(TARGET <PKG-NAME>::<PKG-NAME>
                 PROPERTY INTERFACE_COMPILE_OPTIONS
                     $<$<CONFIG:Release>:${{<PKG-NAME>_COMPILE_OPTIONS_RELEASE_LIST}}>
                     $<$<CONFIG:RelWithDebInfo>:${{<PKG-NAME>_COMPILE_OPTIONS_RELWITHDEBINFO_LIST}}>
                     $<$<CONFIG:MinSizeRel>:${{<PKG-NAME>_COMPILE_OPTIONS_MINSIZEREL_LIST}}>
                     $<$<CONFIG:Debug>:${{<PKG-NAME>_COMPILE_OPTIONS_DEBUG_LIST}}>)

The targets are also transitive. So, if your project depends on a packages ``A`` and ``B``, and at the same time
``A`` depends on ``C``, the ``A`` target will contain automatically the properties of the ``C`` dependency, so
in your `CMakeLists.txt` file you only need to ``find_package(A CONFIG)`` and ``find_package(B CONFIG)``.

.. important::

    Add the ``CONFIG`` option to ``find_package`` so that *module mode* is explicitly skipped by CMake.
    This helps to solve issues when there is for example a ``Find<PKG-NAME>.cmake`` file in CMake's default modules directory
    that could be loaded instead of the ``<PKG-NAME>Config.cmake`` generated by Conan.

You also need to adjust `CMAKE_PREFIX_PATH <https://cmake.org/cmake/help/v3.0/variable/CMAKE_PREFIX_PATH.html>`_ and
`CMAKE_MODULE_PATH <https://cmake.org/cmake/help/v3.0/variable/CMAKE_MODULE_PATH.html>`_ so CMake can locate all
the ``<PKG-NAME>Config.cmake``/``<PKG-NAME>-config.cmake`` files: The ``CMAKE_PREFIX_PATH`` is used by the ``find_package`` and the
``CMAKE_MODULE_PATH`` is used by the ``find_dependency`` calls that locates the transitive dependencies.

The *<PKG-NAME>Targets-.cmake* files use `<PKG-NAME>_BUILD_MODULES_<BUILD-TYPE>` values to include the files using the `include(...)` CMake
directive after the targets are created. This makes functions or utilities exported by the package available for consumers just by setting
`find_package(<PKG-NAME>)` in the *CMakeLists.txt*.

Moreover, this also adjusts `CMAKE_MODULE_PATH` and `CMAKE_PREFIX_PATH` to the values declared by the package in ``cpp_info.buildirs``, so
modules in those directories can be found.

Components
++++++++++

If a recipe uses :ref:`components <package_information_components>`, the targets generated will be ``<PKG-NAME>::<COMP-NAME>`` with the following properties adjusted. Being
``<COMP-NAME>`` the dictionary key used to declare the component or the one declared in ``cpp_info.names["cmake_find_package_multi"]`` or the alternative name declared in
``cpp_info.components["comp_name"].names["cmake_find_package_multi"]`` if specified:

- ``INTERFACE_INCLUDE_DIRECTORIES``: Containing all the include directories of the component.
- ``INTERFACE_LINK_LIBRARIES``: Containing the targets to link the component to (includes component's libraries and dependencies).
- ``INTERFACE_COMPILE_DEFINITIONS``: Containing the definitions of the component.
- ``INTERFACE_COMPILE_OPTIONS``: Containing the compile options of the component.

Moreover, a global target ``<PKG-NAME>::<PKG-NAME>`` will be declared with the following properties adjusted:

- ``INTERFACE_LINK_LIBRARIES``: Containing the list of targets for all the components in the package.

.. important::

    **Name conflicts**: If the name of the global target is the same for different packages, Conan will aggregate into this global target
    all the components from all those different packages. This means that this global target will contain information coming from different
    packages. For the components themselves, a name conflict will result in one of them being inaccessible without further notice.
