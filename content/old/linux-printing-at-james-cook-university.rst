Linux: Printing at James Cook University (or, Fuji Xerox Printers on 64 bit Linux)
##################################################################################
:date: 2010-10-04 15:57
:author: davidjb
:category: Linux
:slug: linux-printing-at-james-cook-university

.. note:: **Update 2016**: this still works on Ubuntu 16.04. Also, I happened
   across `another blog
   <https://robotsfuckyeahalloneword.svbtle.com/fuji-xerox-on-64-bit-linux>`_
   which makes a very valliant effort to get the i386 RPM working on 64-bit
   Linux but then the author resorts to using a VM with 32-bit Ubuntu. The
   following works perfectly on 64-bit natively because you're actually
   building the binaries from source.  Sure, the drivers are from 2006 but
   they're fine.  I guess if it ain't broke...

   Also, this works on Linux and Ubuntu to support all the FX models like
   ApeosPort-II C7500 / C6500 / C5400, DocuCentre-II C7500 / C6500 /
   C5400, ApeosPort-II C4300 / C3300 / C2200DocuCentre-II C4300 / C3300 /
   C2200, ApeosPort-II 7000 / 6000 / 5000, DocuCentre-II 7000 / 6000 / 5000,
   ApeosPort-II 4000 / 3000, DocuCentre-II 4000 / 3000 and anything I've tried
   in between.  In particular, my printer is currently a ApeosPort-V C2275 and
   even follow-me printing works via our print share.

   Alternatively, if you can live without using the FX drivers, the generic
   PostScript PPDs on Ubuntu are fine too.

Printing at JCU is difficult on Linux, given JCU exists as a Windows
environment. The fact that Xerox Postscript drivers for Linux aren't
exactly straight forward to install doesn't help either - they require
custom filter applications to be compiled.

Setting up the drivers
----------------------

#. To make sure you have the right drivers, get this file:
   `fxlinuxprint-src-1.0.1.tar.gz`_, which is the drive source kindly
   obtained from the `Japanese Xerox site`_.
#. Extract the archive somewhere on your hard drive:

   .. code:: bash

       tar -xf fxlinuxprint-src-1.0.1.tar.gz

#. Make sure you've got **build-essential** and **libcups2-dev**
   packages installed (if on Debian/Ubuntu). Other systems (Red Hat,
   CentOS) will require equivalent packages; post if you've done it.

   .. code:: bash

       sudo apt-get install build-essential libcups2-dev

#. CMMI:

   .. code:: bash

       ./configure && make && sudo make install

#. Select this as your PPD when adding your printer as this file:

   .. code:: bash

       /usr/share/cups/model/FujiXerox/en/fxlinuxprint.ppd


Setting up the printer
----------------------

#. Add a new ``Windows printer via SAMBA``
#. Specify the printer address as
   ``smb://AD/print-tsv.jcu.edu.au/DB017-51-Colour``, where ``DB017-51-Colour``
   is your printer name (you'll probably need to ask someone or browse
   the machine at that hostname to find out; on Linux you can use
   ``smbclient -L print-tsv.jcu.edu.au -U [your user ID] -W AD``)
#. Enter your authentication details (password)
#. Press ``Verify`` to make sure it works, and then click ``Forward``
#. Under ``Choose Driver`` (or when selecting a Make and Model), make sure
   you choose to ``Provide PPD file``, and use the one that you've just
   installed using the instructions above.
   (``/usr/share/cups/model/FujiXerox/en/fxlinuxprint.ppd``).
#. Click ``Forward`` when done.
#. Add the rest of your printer's details and click ``Apply``.

Try and print something and it should be good now.  If your machine is like
mine, you may find that your pages get ``Held for authentication`` and you
don't get prompted to enter a user name and password -- if this is the case,
then just open the ``Printers`` application, view your printer's print queue,
right-click on your job and ``Authenticate`` it.

.. _fxlinuxprint-src-1.0.1.tar.gz: |filename|../files/fxlinuxprint-src-1.0.1.tar.gz
.. _Japanese Xerox site: http://www.fujixerox.co.jp/download/apeosport/download/c4300series/linux_module.html
