Requirements
------------

``imexam`` currently provides display support two viewers: DS9 and Ginga. The default is for imexam to start up it's own DS9 process and shut it down nicely upon exit. A Ginga widget using an HTML5 backend is also available as a viewer. The package is designed so that it may easily accept other display devices in the future.

The ``imexam`` library can be used standalone, without a viewer, to create the plots which are available in the interactive sessions by importing the plotting object and feeding the functions your data with x,y coordinates for the plots. It can also be used within the Jupyter notebook framework with either DS9 or the HTML5 backend for viewing. In either case, the images and plots are saved inside the notebook in conjunction with the nbAgg matplotlib backend.

.. note:: For DS9, it is important to know if you have XPANS installed on your machine and available through your PATH if you plan to use the nameserver functionality. XPANS is the XPA Name Server, it keeps track of all the open socket connections for DS9 and provides a reference for their names. If you DO NOT have XPANS installed, then ``imexam`` will still work, but you should either start the DS9 window after importing ``imexam`` through the imexam.connect() interface, OR after you start DS9 from the shell, make note of the ``XPA_METHOD`` address in the File->Information dialog and use that in your call to connect: window=imexam.connect(XPA_METHOD) so that communication with the correct socket is established.

In order to use the Ginga widget display you must have Ginga installed. More information about Ginga can be found in its package documentation: http://ginga.readthedocs.org/en/latest/. If you are using Python 3 you should also install Pillow which will aid in the image display. The Ginga documentation will
tell you of any of it's other dependencies. If you install Ginga you will have access to another display tool for your images and data, the HTML5 widget. You can find the source code on GitHub, but you can also install it with ``pip`` or ``conda``.

You can access this help file on your locally installed copy of the package by using the imexam.display_help() call after import. This will display the help in your web browser.

.. note:: All information returned from this module should be considered an estimate of an actual refined result,  more precise analysis of the data should be performed for verification before publication.


How to Install
--------------

These are some tips on installing the package, or tracking down problems you might be having during or after installation.

``imexam`` can be installed from the source code in the normal python fashion after downloading it from the git repo::

    python setup.py install


``imexam`` can also be installed using pip or conda, and is included in the Astroconda distribution from STScI:

    pip install imexam
    pip install --upgrade imexam #if you already have an older version installed
    
    conda install imexam



If you want to have access to the photometry features of the ``imexam`` analysis, download and install ``photutils`` - another of the astropy associated packages. The full list of astropy packages can be found here: https://github.com/astropy
If ``photutils`` is not installed, ``imexam`` should issue a nice statement saying that the photometry options are not available upon import, and any time an analysis key is pressed during the imexam() function loop which requires ``photutils`` to render a result.


Usage
-----

``imexam`` displays plots using matplotlib, if you find that no windows are popping up after installation it's probably the backend that was loaded. One quick way to get things started is to load ipython  and use the %matplotlib magic, this will make sure the proper display backend loads when matplotlib is imported::

    >ipython
    >%matplotlib
    >import imexam

Matplotlib magic should also be used inside the Jupyter notebook or proper interaction with the plots. Before importing ``imexam`` into the notebook, specify the ``nbAgg`` backend if you wish to save your plots into the notebook itself. Otherwise you can use the standard magic.


``imexam`` is a class based library. The user creates an object which is tied to a specific image viewing window, such as a DS9 window. In order to interact with multiple  windows the user must create multiple objects. Each object stores all the relevent information about the window and data with which it is associated.

For example, in order to open a new DS9 window and use the object "viewer" to control it, you would issue the command:

::

    viewer=imexam.connect()

The "viewer" object now has associated methods to view, manipulate and analyze data in the DS9 session. When you start the connection, you also have the option of specifying a currently open DS9 window using the target keyword. This keyword can contain the name, the actual text name that you gave the window, or the address of the window.  The address of the window can be found in the File->XPA->Information menu item,  is stored as ``XPA_METHOD``, and is of the form "82a7e75f:58576" for INET sockets, and a file path for local sockets. The following is an example of connecting to an already active DS9 window which was started outside of ``imexam``::


    viewer=imexam.connect("82a7e75f:58576")

    or

    viewer=imexam.connect("my_window_title")


When ``imexam`` starts up a DS9 window itself, it will create a local socket by default, even though the default socket type for DS9 is INET. However, ``imexam`` will first check to see if ``XPA_METHOD`` was set in your environment and default to that option. If you are experiencing problems, or you don't have an internet connection (the two might be related because the XPA structures INET sockets with an ip address), you can set your environment variable ``XPA_METHOD`` to ``local`` or ``localhost``. This will cause ``imexam`` to start a local(unix) socket which will show an ``XPA_METHOD`` that is a filename on your computer. ``imexam`` defaults to a local socket connection to allow for users who do not have the XPA installed on their machine or available on their PATH.

The full XPA source code is maintained as a submodule to the ``imexam` package. If you don't have the XPA on your path, simply point it to that location, or copy the xpans executable to the location of your choice, and make sure you update your PATH. Any time DS9 is started it will start up the xpa nameserver automatically. Then all the xpans query options will be available through ``imexam`` (such as imexam.list_active_ds9()).  ``imexam`` itself uses Cython wrappers around the ``get`` and ``set`` methods from the XPA for it's communication which is why the fully installed XPA is not necessary.

If you wish to open multiple DS9 windows outside of ``imexam``, then it's recommended that you give each a unique name. If you've forgotten which window had which name, you can look in the same XPA info menu and use the ``XPA_NAME`` specified there. If you haven't given them a unique name, you can list the available windows using imexam.list_active_ds9() (as long as XPANS is running) and specify their unique address.

``imexam`` will attempt to find the current location of the DS9 executable by default, but you may also supply the path to the DS9 executable of your choice using the path keyword when you call connect. The fully optional calling sequence is:


::

    imexam.connect(target="",path=None,viewer="ds9",wait_time=10)

    Where target is the name of the ds9 window that is already running, path is the location of the ds9 executable, viewer is the name of the viewer to use (ds9 is the only one which is currently activated), and wait_time is the time to wait to establish a connection to the socket before exiting the process.

If it seems like the ds9 window is opening or hanging, there could be few things going on:


    * ``imexam`` will default to a local unix connection for the XPA to help with users who are not connected to the internet. The default DS9 connection is INET. However, it will first check your environment variable ``XPA_METHOD`` and preferably use that instead. If you don't have an internet connection, check this environment variable.
    * If things seem in order, it's possible that your machine is waiting for X11 to start up, give it time to start, or when you call ``imexam`` increase the wait time sufficiently; you can do this by specifying "wait_time=60" when you open your viewing object with connect(). The 60 here is an example of the number of seconds ``imexam`` should wait before returning a connection error.
    * Next, check that the path to the DS9 executable is somewhere on your path and that it has not been aliased to something else. You can check this from any terminal window by trying to start DS9. You can also use the unix "which ds9" command to return the full path to the executable, as well as "ls -al ds9" to return the full path and any soft links which might have been established.



In order to return a list of the current DS9 windows that are running, issue the command:

::

    imexam.list_active_ds9()


If you are using the Ginga  widget, the interaction with the ``imexam`` code stays the same, you simply specify that you would like to use Ginga in the call to connect:

::

    viewer=imexam.connect(viewer='ginga')



"ginga" tells ``imexam`` that you'd like to use the Ginga widget with the HTML5 background.


In order to turn logging to a file on, issue the command: window.setlog(). The log will be saved to the default filename imexam_session.log in the current directory unless you give it another filename to use.
Here's an example of how that might work:

::

    import imexam
    window=imexam.connect('ds9')
    window.setlog() <-- turns on logging with default filename
    window.imexam() <-- all output will be logged to the file and displayed on the screen
    window.setlog(on=False) <-- turns off logging to file
    window.setlog(filename='my_other_log.txt') <-- turns on logging and sets the save filename


The log will look something like this, you can see it contains a mention of the command used along with the results

::

    gauss_center
    xc=812.984250   yc=706.562612

    aper_phot
    x       y       radius  flux    mag(zpt=25.00)  sky     fwhm
    812.98  706.56  5       1288669.29      9.72    11414.53        4.83

    show_xy_coords
    813.5 706.625

    gauss_center
    xc=812.984250   yc=706.562612

    gauss_center
    xc=239.856464   yc=233.444783

    aper_phot
    x       y       radius  flux    mag(zpt=25.00)  sky     fwhm
    239.86  233.44  5       126601.26       12.24   11574.32        -12.67

    show_xy_coords
    253.0 234.75

    gauss_center
    xc=239.856464   yc=233.444783


More detailed examples can be found in the examples section of this documentation.


.. note:: More information on DS9 can be found at: http://ds9.si.edu/site/Home.html


Common Problems
---------------

You're getting the following error statement when you try to connect() to a DS9 window, or display an image:

::

    XpaException: Unknown XPA Error : XPAGet returned 0!


You can first try using local unix sockets by setting your environment variable ``XPA_METHOD`` to local:

::

    setenv XPA_METHOD local


That will create local unix file sockets for communication with ds9. If that doesn't solve the problem, see if your path includes the location of xpans, the XPA name server. If you have it installed, but it's not on your path, put it there.
You can also compile and install the XPA software included with ``imexam`` by editing the setup.py file:


    * Uncomment the section towards the bottom of the file which instructs you to uncomment to compile the full XPA
    * Also uncomment the last line with CMDCLASS and add that line inside the parenthesis for the setup command just above.
    * Now you can reinstall ``imexam`` the normal way and it will also build the XPA executables and store them in cextern/
    * Make sure the path to the executables is on your system PATH


Now you can start an ipython window, import ``imexam`` and try starting a new DS9 connection. If this still doesn't solve your problem, send email to the developers or open an issue in github.


If you are using Ginga and the plotting window seems to block, check to see if you've specified the QT backend in any of your matplotlib defaults and try turning it off:

::

    matplotlib.use('Qt4Agg')  <-- remove this and see if it helps
