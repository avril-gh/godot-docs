.. _doc_compiling_for_uwp:

Compiling for Universal Windows Platform
========================================

.. highlight:: shell

Requirements
------------

-  SCons (see :ref:`doc_compiling_for_windows` for more details).
-  Visual Studio 2015 Update 2. It may work with earlier versions. See
   :ref:`doc_compiling_for_windows` about the caveats of installing it
   and the various prompts.
-  Windows 10 SDK (can be selected in Visual Studio installation).
-  `ANGLE source <https://github.com/Microsoft/angle>`__. Use the 
   ``ms_master`` (default) branch. Keep it in a path without spaces to
   avoid problems.

Compiling
---------

You need to open a proper Visual Studio prompt for the target architecture
you want to build. Check :ref:`doc_compiling_for_windows` to see how these
prompts work.

There are three target architectures for UWP: x86 (32-bits), x64 (64-bits)
and ARM (32-bits). For the latter, you can run ``vcvarsall.bat`` with
``x86_arm`` or ``amd64_arm`` as argument to set the environment.

Set the ``ANGLE_SRC_PATH`` to the directory where you downloaded the ANGLE
source code. The build process will also build ANGLE to produce the
required DLLs for the selected architecture.

Once you're set, run the SCons command similarly to the other platforms::

    C:\godot>scons platform=uwp

Creating UWP export templates
-----------------------------

To export using the editor you need to properly build package the templates.
You need all three architectures with ``debug`` and ``release`` templates to
be able to export.

Open the command prompt for one architecture and run SCons twice (once for
each target)::

    C:\godot>scons platform=uwp target=release_debug
    C:\godot>scons platform=uwp target=release

Repeat for the other architectures.

In the end your ``bin`` folder will have the ``.exe`` binaries with a name
like ``godot.uwp.opt.debug.32.x86.exe`` (with variations for each
target/arch).

Copy one of these to ``tools/dist/uwp_template`` inside the Godot source
folder and rename the binary to ``godot.uwp.exe``. From the ANGLE source,
under ``winrt/10/src/Release_%arch%`` (where ``%arch%`` can be ``Win32``,
``x64`` or ``ARM``), get the ``libEGL.dll`` and the ``libGLESv2.dll``,
putting them along with the executable.

Add the files in the ``uwp_template`` folder to a ZIP. Rename the resulting
Zip according to the target/architecture of the template::

    uwp_x86_debug.zip
    uwp_x86_release.zip
    uwp_x64_debug.zip
    uwp_x64_release.zip
    uwp_arm_debug.zip
    uwp_arm_release.zip

Move those templates to the ``templates`` folder in Godot settings path. If
you don't want to replacet the templates, you can set the "Custom Package"
property in the export window.

Running UWP apps with Visual Studio
-----------------------------------

If you want to debug the UWP port or simply run your apps without packaging
and signing, you can deploy and launch them using Visual Studio. It might be
the easiest way if you are testing on a device such as a Windows Phone or an
Xbox One.

Within the ANGLE source folder, open ``templates`` and double-click the
``install.bat`` script file. This will install the Visual Studio project
templates for ANGLE apps.

If you have not built Godot yet, open the ``winrt/10/src/angle.sln`` solution
from the ANGLE source and build it to Release/Win32 target. You may also need
to build it for ARM if you plan to run on a device. You can also use MSBuild if
you're comfortable with the command line.

Create a new Windows App project using the "App for OpenGL ES
(Windows Unversal)" project template, which can be found under the
``Visual C++/Windows/Universal`` category.

This is a base project with the ANGLE dependencies already set up. However, by 
default it picks the debug version of the DLLs which usually have a very poor
performance. So in the "Binaries" filter, click in each of the DLLs there
and in the "Properties" window and change the relative path from
``Debug_Win32`` to ``Release_Win32`` (or ``Release_ARM`` for devices).

In the same "Binaries" filter, select "Add > Existing Item" and point to the
Godot executable for UWP you have. In the "Properties" window, set "Content"
to ``True`` so it's included in the project.

Right-click the ``Package.appxmanifest`` file and select "Open With... > XML
(Text) Editor". In the ``Package/Applications/Application`` element, replace
the ``Executable`` attribute from ``$targetnametoken$.exe`` to
``godot.uwp.exe`` (or whatever your Godot executable is called). Also change
the ``EntryPoint`` attribute to ``GodotUWP.App``. This will ensure that
the Godot executable is correctly called when the app starts.

Create a folder (*not* a filter) called ``game`` in your Visual Studio project
folder and there you can put either a ``data.pck`` file or your Godot project
files. After that, make sure to include it all with the "Add > Existing Item"
command and set their "Content" property to ``True`` so they're copied to the
app.

To ease the workflow, you can open the "Solution Properties" and in the
"Configuration" section untick the "Build" option for the app. You still have
to build it at least once to generate some needed files, you can do so by
right-clicking the project (*not* the solution) in the "Solution Explorer" and
selecting "Build".

Now you can just run the project and your app should open. You can use also
the "Start Without Debugging" from the "Debug" menu (Ctrl+F5) to make it
launch faster.
