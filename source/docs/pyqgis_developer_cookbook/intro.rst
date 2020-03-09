.. _introduction:

************
Introduction
************

Bu dökümanda aynı anda eğitim ve referans kaynağı olması hedeflenmiştir. Tüm kullanım senaryolarını gösteremesekde genel kullanım yöntemlerini göstermeyi hedefledik.

.. contents::
   :local:

Python desteği ilk olarak QGIS 0.9 versiyonunda başladı.
QGIS Masaüstü versiyonunda Python kullanmanın birçok yolu vardır.
(bu yöntemler):

* QGIS içinde Python konsolundan komutlar verilebilir
* Eklentiler oluşturarak ve kullanarak
* QGIS programını çalıştırdığınız anda otomatik çalışacak Python kodları ile
* QGIS API'si kullanan özelleşmiş programlar yazarak dışarıdan

Python eklentileri ayrıca QGIS Server versiyonunda çalışabilir, eklentilere erişim için (bkz. :ref:`server_plugins`)
Python eklentileri gömülü QGIS Server Python programndan çalıştırılabilir.

.. index:: API

Referans sınıf ve kütüphanelere :api:`QGIS API <>` linkinden bakabilirsiniz. 
:pyqgis:`Pythonic QGIS API (pyqgis) <>` arayüzü,  C++ API eklentisi ile neredeyse aynıdır.

A good resource for learning how to perform common tasks is to
download existing plugins from the
`plugin repository <https://plugins.qgis.org/>`_ and examine their
code.

.. index::
  pair: Python; Console

.. _pythonconsole:

Python Konsoluda Kodlama
===============================

QGIS programı, kodlama için bir :ref:`Python console <console>` içerir.
Konsol :menuselection:`Plugins --> Python Console` menüsünden açılabilir:

.. figure:: img/console.png
   :align: center
   :width: 40em

   QGIS Python konsolu

Yukarıdaki ekran görüntüsündeki kodda katman listesinde seçili katmanı bir değişkene atamayı, id değerini, bir vektör katmanı ise kayıtlı nesne sayısını göstermeyi içeriyor. 
QGIS program arayüzü ile etkileşim için :data:`iface` değişkeni kullanılır, :class:`QgisInterface <qgis.gui.QgisInterface>` sınıfındadır. :data:`iface` ile çizim alanına, menülere, araç kutularına ve diğer QGIS program parçalarına ulaşabilirsiniz.

Kullanıcı kolaylığı için, aşağıdaki kodlar konsol açılır açılmaz çalıştırılır. (ileride farklı komutların otomatik çalıştırılabilmesini ayarlamak mümkün olacaktır.)

::

  from qgis.core import *
  import qgis.utils

Konsolu sık ullananlar için klavye kısayolu ayrlamak mümkündür. (menüde
:menuselection:`Settings --> Keyboard shortcuts...`)

.. index:: Python; Plugins

Python Eklentileri
==============

QGIS işlevleri eklentilerle genişletilebilir, yeni özellikler eklenebilir. C++ eklentilerine göre Python eklentilerinin avantajı farklı platformlarda kolayca çalışabilmesi, derlenebilmesidir.

Python desteği verilmeye başlandığından bu yana birçok eklenti yazılmıştır. Eklenti yükleyici, kullanıcıların eklentileri kolayca yüklemesini, güncellemesini, kaldırmasını sağlar.
`Python Plugins <https://plugins.qgis.org/>`_ sayfasında detaylı eklenti bilgilerine ve geliştirme ortamı bilgilerine ulaşabilirsiniz. 

Python içinde eklenti oluşturmak kolaydır, detaylı bilgi için :ref:`developing_plugins`
sayfasına bakınız..

.. note::

    Python eklentileri, ayrıdca QGIS sunucu versiyonu için de mevcuttur. Detaylı bilgi için :ref:`server_plugins` sayfasına bakınız.


.. index::
  pair: Python; Startup

QGIS başlangıcında Python kodunu çalıştırma
====================================

Program açıldığında otomatik kod çalıştırmanın iki farklı yolu vardır. 

1. startup.py kodu oluşturarak.

2. ``PYQGIS_STARTUP`` ortam değişkenine bir py dosyası atamak. 


 :file:`startup.py` dosyası
----------------------------

QGIS her başladığında Python ana çalışma dizinine bakar

* Linux: :file:`.local/share/QGIS/QGIS3`
* Windows: :file:`AppData\\Roaming\\QGIS\\QGIS3`
* macOS: :file:`Library/Application Support/QGIS/QGIS3`

:file:`startup.py` dosyasını arar. Dosya mevcutsa programın içinde gelen birleşik python modülüyle çalıştırır.

.. note:: Varsayılan QGIS program yolu işletim sistmine göre değişiklik gösterir. Python konsolunu açın ve 
  ``QStandardPaths.standardLocations(QStandardPaths.AppDataLocation)`` kodunu çalıştırın, programın kurulu olduğu ana çalışma dizinlerinin listesini konsola yazdıracaktır.

.. index::
  pair: Environment; PYQGIS_STARTUP

The PYQGIS_STARTUP ortam değişkeni
---------------------------------------

``PYQGIS_STARTUP`` ortam değişkenine bir dosya yolu ataması yaparak QGIS ekrana gelmeden önce atanan py dosyasının çalışmasını sağlayabilirsiniz. 

Bu kod QGIS başlamadan çalışır. 
This method is very useful for cleaning
sys.path, which may have undesireable paths, or for isolating/loading
the initial environment without requiring a virtual environment, e.g.
homebrew or MacPorts installs on Mac.

.. index::
  pair: Python; Custom applications
  pair: Python; Standalone scripts

.. _pythonapplications:

Python Uygulamaları
===================

It is often handy to create  scripts for automating processes.
With PyQGIS, this is perfectly possible --- import
the :mod:`qgis.core` module, initialize it and you are ready for the
processing.

Or you may want to create an interactive application that uses
GIS functionality --- perform measurements, export a map as PDF, ...
The :mod:`qgis.gui` module provides various GUI
components, most notably the map canvas widget that can be
incorporated into the application with support for zooming, panning
and/or any further custom map tools.

PyQGIS custom applications or standalone scripts must be configured to
locate the QGIS resources, such as projection information and providers
for reading vector and raster layers. QGIS Resources are
initialized by adding a few lines to the beginning of your application
or script. The code to initialize QGIS for custom applications and
standalone scripts is similar. Examples of each are provided
below.

.. note::

     Do *not* use :file:`qgis.py` as a name for your script.
     Python will not be able to import the bindings as the script's
     name will shadow them.

.. _standalonescript:

PyQGIS bağımsız kodları kullanma
----------------------------------

Kodun başlangıcında QGIS kaynak kodlarına çağırma: 

::

  from qgis.core import *

  # QGIS yükleme konumunu gösterme
  QgsApplication.setPrefixPath("/path/to/qgis/installation", True)

  # QgsApplication referansını oluşturun. İkinci argüman olarak
  # False ayarı GUI arayüzünü devre dışı bırakır. 
  qgs = QgsApplication([], False)

  # Kodu başlat
  qgs.initQgis()

  # Kodunuzu buraya yazın. KAtmanları yükleyin, işlem Write your code here to load some layers, use processing
  # algoritmalarını işletin, v.s.

  # Son olarak, exitQgis() fonksiyonunu çağırın, 
  # hafızayı boşaltma işlemini tetikleyin. 

  qgs.exitQgis()

Öncelikle :mod:`qgis.core` modülünü ayarlayın ve çağırın. 
Modül çalışma konumu ayarlaması için QGIS in bilgisayarınızdaki yükleme yerini bulun. 
Yükleme yeri bulma kodu :meth:`setPrefixPath <qgis.core.QgsApplication.setPrefixPath>` yöntemi ile çağrılır. 
:meth:`setPrefixPath <qgis.core.QgsApplication.setPrefixPath>` metodunun 
ikinci argümanını ``True`` olarak ayarlayarak ön ayarlı yolu kulanacağınızı belirtirsiniz. 

QGIS yükleme konumu işletim sistemine bağlı olarak değişir. 
Yükleme konumunu bulmanın en kolay yolu :ref:`pythonconsole` içinden 
``QgsApplication.prefixPath()`` komutunu çalıştırarak çıktıya bakmaktır.

After the prefix path is configured, we save a reference to
``QgsApplication`` in the variable ``qgs``. The second argument is set
to ``False``, specifying that we do not plan to use the GUI since
we are writing a standalone script. With ``QgsApplication``
configured, we load the QGIS data providers and layer registry by
calling the ``qgs.initQgis()`` method. With QGIS initialized, we are
ready to write the rest of the script. Finally, we wrap up by calling
``qgs.exitQgis()`` to remove the data providers and layer registry
from memory.


PyQGIS kodlarını özel bir program için kullanma
-----------------------------------

The only difference between :ref:`standalonescript` and a custom PyQGIS
application is the second argument when instantiating the ``QgsApplication``.
Pass ``True`` instead of ``False`` to indicate that we plan to
use a GUI.

::

  from qgis.core import *

  # Supply the path to the qgis install location
  QgsApplication.setPrefixPath("/path/to/qgis/installation", True)

  # Create a reference to the QgsApplication.
  # Setting the second argument to True enables the GUI.  We need
  # this since this is a custom application.

  qgs = QgsApplication([], True)

  # load providers
  qgs.initQgis()

  # Write your code here to load some layers, use processing
  # algorithms, etc.

  # Finally, exitQgis() is called to remove the
  # provider and layer registries from memory
  qgs.exitQgis()


Now you can work with the QGIS API - load layers and do some processing or fire
up a GUI with a map canvas. The possibilities are endless :-)


.. index::
  pair: Custom applications; Running

Running Custom Applications
---------------------------

You need to tell your system where to search for QGIS libraries and
appropriate Python modules if they are not in a well-known location -
otherwise Python will complain::

  >>> import qgis.core
  ImportError: No module named qgis.core

This can be fixed by setting the ``PYTHONPATH`` environment variable. In
the following commands, ``<qgispath>`` should be replaced with your actual
QGIS installation path:

* on Linux: :command:`export PYTHONPATH=/<qgispath>/share/qgis/python`
* on Windows: :command:`set PYTHONPATH=c:\\<qgispath>\\python`
* on macOS: :command:`export PYTHONPATH=/<qgispath>/Contents/Resources/python`

Now, the path to the PyQGIS modules is known, but they depend on
the ``qgis_core`` and ``qgis_gui`` libraries (the Python modules serve
only as wrappers). The path to these libraries may be unknown to the
operating system, and then you will get an import error again (the message
might vary depending on the system)::

  >>> import qgis.core
  ImportError: libqgis_core.so.3.2.0: cannot open shared object file:
    No such file or directory

Fix this by adding the directories where the QGIS libraries reside to
the search path of the dynamic linker:

* on Linux: :command:`export LD_LIBRARY_PATH=/<qgispath>/lib`
* on Windows: :command:`set PATH=C:\\<qgispath>\\bin;C:\\<qgispath>\\apps\\<qgisrelease>\\bin;%PATH%`
  where ``<qgisrelease>`` should be replaced with the type of release
  you are targeting (eg, ``qgis-ltr``, ``qgis``, ``qgis-dev``)

These commands can be put into a bootstrap script that will take care of
the startup. When deploying custom applications using PyQGIS, there are
usually two possibilities:

* require the user to install QGIS prior to installing your
  application. The application installer should look for default locations
  of QGIS libraries and allow the user to set the path if not found. This
  approach has the advantage of being simpler, however it requires the user
  to do more steps.

* package QGIS together with your application. Releasing the application
  may be more challenging and the package will be larger, but the user will
  be saved from the burden of downloading and installing additional pieces
  of software.

The two deployment models can be mixed.  You can provide a standalone
applications on Windows and macOS, but for Linux leave the installation of
GIS up to the user and his package manager.

Technical notes on PyQt and SIP
===============================

We've decided for Python as it's one of the most favoured languages for
scripting. PyQGIS bindings in QGIS 3 depend on SIP and PyQt5.
The reason for using SIP instead of the more widely used SWIG is that the
QGIS code depends on Qt libraries. Python bindings for Qt (PyQt) are
done using SIP and this allows seamless integration of PyQGIS with
PyQt.
