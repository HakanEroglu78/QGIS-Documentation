.. index:: Plugins; User interaction

***************************
Kullanıcı iletişim arayüzü
***************************

.. contents::
   :local:

Bu bölüm kullanıcı ile iletişim kurmamızı sağlayan metod ve bileşenlerin kullanımını içerir.

Mesaj gösterme. QgsMessageBar sınıfı.
=========================================

Mesaj kutuları kullanarak mesajlarımızı göstermek kullanıcı açısından kötü bir deneyim oluşturur. Bunun yerine uyarı ve hata mesajlarını bilgi çubuğu, mesaj satırı bölmesinde göstermek QGIS için daha mantıklı bir yöntemdir. Toplu işlemlerde yüzlerce hata ve uyarı mesajı oluşabilir.

QGIS arayüzünde mesaj satırı kısmını kullanarak mesaj göstermek için kullanılacak kod örneği:

.. code-block:: python

  from qgis.core import Qgis
  iface.messageBar().pushMessage("Error", "I'm sorry Dave, I'm afraid I can't do that", level=Qgis.Critical)


.. figure:: img/errorbar.png
   :align: center
   :width: 40em

   QGIS Mesaj çubuğu

Mesajların belirli bir süre gösterilip ardından mesaj çubuğunun kaybolmasını sağlamak için süre sınırı koyabilirsiniz.

.. code-block:: python

    iface.messageBar().pushMessage("Ooops", "The plugin is not working as it should", level=Qgis.Critical, duration=3)


.. figure:: img/errorbar-timed.png
   :align: center
   :width: 40em

   QGIS Mesaj çubuğu, zaman sınırlaması ile

Yukarıdaki örnekte hata mesajı görülmekte, fakat ``level`` parametresi uyarı veya bilgi mesajı tipi olabilir. :class:`Qgis.MessageLevel <qgis.core.Qgis.MessageLevel>` seçenekleri kullanılabilir. 4 farklı seviye öntanımlıdır:

0. Bilgi
1. Uyarı
2. Kritik
3. Başarılı

.. figure:: img/infobar.png
   :align: center
   :width: 40em

   QGIS Mesaj çubuğu (bilgi)

Mesaj çubuğuna araçlar eklenebilir, örneğin daha fazla bilgi göstermek için bir buton

.. code-block:: python

    def showError():
        pass

    widget = iface.messageBar().createMessage("Missing Layers", "Show Me")
    button = QPushButton(widget)
    button.setText("Show Me")
    button.pressed.connect(showError)
    widget.layout().addWidget(button)
    iface.messageBar().pushWidget(widget, Qgis.Warning)


.. figure:: img/bar-button.png
   :align: center
   :width: 40em

   Butonlu bir QGIS Mesaj çubuğu

Bu şekilde mesaj çubuğunu kendi mesaj kutunuzu göstermek için kullanabilirsiniz yada mesaj kutusu göstermek zorunlu değilse kullanılabilir. 

.. code-block:: python

    class MyDialog(QDialog):
        def __init__(self):
            QDialog.__init__(self)
            self.bar = QgsMessageBar()
            self.bar.setSizePolicy( QSizePolicy.Minimum, QSizePolicy.Fixed )
            self.setLayout(QGridLayout())
            self.layout().setContentsMargins(0, 0, 0, 0)
            self.buttonbox = QDialogButtonBox(QDialogButtonBox.Ok)
            self.buttonbox.accepted.connect(self.run)
            self.layout().addWidget(self.buttonbox, 0, 0, 2, 1)
            self.layout().addWidget(self.bar, 0, 0, 1, 1)
        def run(self):
            self.bar.pushMessage("Hello", "World", level=Qgis.Info)

    myDlg = MyDialog()
    myDlg.show()

.. figure:: img/dialog-with-bar.png
   :align: center
   :width: 40em

   QGIS, özel bileşenli mesaj çubuğu


İlerleme durumu gösterme
================

İlerleme durum çubukları QGIS mesaj çubuğu içinde gösterilebilir. İşte burada konsolda deneyebileceğiniz örnek bir kod.

.. code-block:: python

    import time
    from qgis.PyQt.QtWidgets import QProgressBar
    from qgis.PyQt.QtCore import *
    progressMessageBar = iface.messageBar().createMessage("Doing something boring...")
    progress = QProgressBar()
    progress.setMaximum(10)
    progress.setAlignment(Qt.AlignLeft|Qt.AlignVCenter)
    progressMessageBar.layout().addWidget(progress)
    iface.messageBar().pushWidget(progressMessageBar, Qgis.Info)

    for i in range(10):
        time.sleep(1)
        progress.setValue(i + 1)

    iface.messageBar().clearWidgets()

Ayrıca entegre ilerleme durum çubuğunu kullanabilirsiniz. İşte sonraki örnek kod:

.. code-block:: python

 vlayer = QgsProject.instance().mapLayersByName("countries")[0]

 count = vlayer.featureCount()
 features = vlayer.getFeatures()

 for i, feature in enumerate(features):
     # do something time-consuming here
     print('') # printing should give enough time to present the progress

     percent = i / float(count) * 100
     # iface.mainWindow().statusBar().showMessage("Processed {} %".format(int(percent)))
     iface.statusBarIface().showMessage("Processed {} %".format(int(percent)))

 iface.statusBarIface().clearMessage()


Loglama
=======

QGIS loglama sistemini kullanarak kodunuzun işletilme sürecini kaydettirebilirsiniz. 

.. code-block:: python

 # You can optionally pass a 'tag' and a 'level' parameters
 QgsMessageLog.logMessage("Your plugin code has been executed correctly", 'MyPlugin', level=Qgis.Info)
 QgsMessageLog.logMessage("Your plugin code might have some problems", level=Qgis.Warning)
 QgsMessageLog.logMessage("Your plugin code has crashed!", level=Qgis.Critical)

.. warning::

Python ``print`` komutunu kullanmak çoklu işlemlerin çalışmasında sorun oluşturur. Aynı şekilde **expression functions**, **renderers**, **symbol layers** ve **Processing algorithms** (ve diğer) fonksiyonlar içinde geçerlidir. Bu durumda her zaman güvenli sınıflar olan (:class:`QgsLogger <qgis.core.QgsLogger>` veya :class:`QgsMessageLog <qgis.core.QgsMessageLog>`) kullanmalısınız.


.. note::

   :class:`QgsMessageLog <qgis.core.QgsMessageLog>` sınıfının çıktılarını  :ref:`log_message_panel` içerisinde görebilirsiniz.

.. note::

 * :class:`QgsLogger <qgis.core.QgsLogger>` mesajlaşma ve yazılımcılar için hata arama içindir (hatalı bir kodun tetiklendiğinden şüphelendiğiniz durumlar gibi)
 * :class:`QgsMessageLog <qgis.core.QgsMessageLog>` sysadmin grubu için problemleri çözmesi amacıyla mesajlar gösterir (sysadmin' lerin ayarları düzelmesi gibi)
