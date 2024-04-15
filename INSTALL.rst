Installation instructions for Xtables-addons
============================================

Xtables-addons uses the well-known configure(autotools) infrastructure
in combination with the kernel's Kbuild system.

.. code-block:: sh

	$ ./configure
	$ make
	# make install


Supported configurations for this release
=========================================

* iptables >= 1.6.0

* kernel-devel >= 5.4
  with prepared build/output directory

  * ``CONFIG_NF_CONNTRACK``

  * ``CONFIG_NF_CONNTRACK_MARK`` enabled =y or as module (=m)

  * ``CONFIG_CONNECTOR`` y/m if you wish to receive userspace
    notifications from pknock through netlink/connector (if not, edit the
    ``mconfig`` file in Xtables-addons to exclude pknock)

  * ``CONFIG_TEXTSEARCH_BM`` y/m if you wish to use xt_ipp2p (if not, edit the
    ``mconfig`` file in Xtables-addons to exclude ipp2p)

(Use xtables-addons-1.x if you need support for Linux < 3.7.
Use xtables-addons-2.x if you need support for Linux < 4.15.
Use xtables-addons<3.19 if you need support for Linux <=4.16.)
Note: xtables-addons regularly fails to build with patched-to-death
kernels like on RHEL or SLES because the API does not match
LINUX_KERNEL_VERSION anymore.


Selecting extensions
====================

You can edit the ``mconfig`` file to select what modules to build and
install. By default, all modules are enabled.


Configuring and compiling
=========================

.. code-block:: sh

	./configure [options]

``--without-kbuild``
	Deactivate building kernel modules, and just do userspace parts.

``--with-kbuild=``
	Specifies the path to the kernel build output directory. We need
	it for building the kernel extensions. It defaults to
	``/lib/modules/$(running version)/build``, which usually points to
	the right directory. (If not, you need to install something.)

	For RPM building, it should be ``/usr/src/linux-obj/...``
	or whatever location the distribution makes use of.

``--with-xtlibdir=``
	Specifies the path to where the newly built extensions should
	be installed when ``make install`` is run. The default is to
	use the same path that Xtables/iptables modules use, as
	determined by ``pkg-config xtables --variable xtlibdir``.
	Thus, this option normally does *not* need to be specified
	anymore, even if your distribution put modules in a strange
	location.

If you want to enable debugging, use

.. code-block:: sh

	./configure CFLAGS="-ggdb3 -O0"

(``-O0`` is used to turn off instruction reordering, which makes debugging
much easier.)

To make use of a libxtables that is not in the default path, either

a) append the location of the pkg-config files like:

   .. code-block:: sh

   	PKG_CONFIG_PATH=/usr/local/lib/pkgconfig

   (Assuming that files have been installed), or,

b) override the pkg-config variables, for example:

   .. code-block:: sh

   	./configure libxtables_CFLAGS="-I../iptables/include" \
   	libxtables_LIBS="-L../iptables/.libs \
   	-Wl,-rpath,../iptables/.libs -lxtables"

   (Use this in case you wish to use it without having to
   run ``make install``. This is because the ``libxtables.pc`` pkgconfig
   file in ../iptables would already point to e.g. ``/usr/local``.)


Build-time options
==================

``V=``
	This variable controls the verbosity of make commands:

	* ``V=0``: "silent" (output filename)

	* ``V=1``: "verbose" (entire gcc command line)


Note to distribution packagers
==============================

Except for ``--with-kbuild``, distributions should not have a need to
supply any other flags (besides ``--prefix=/usr`` and perhaps
``--libdir=/usr/lib64``, etc.) to configure when all prerequired packages
are installed. If *iptables-devel* is installed, necessary headers should
already be in ``/usr/include``, so that overriding ``PKG_CONFIG_PATH``,
``libxtables_CFLAGS`` and ``libxtables_LIBS`` variables should not be needed.
