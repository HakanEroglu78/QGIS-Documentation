.. index:: Plugins; User interaction

***************************
Kullanıcı iletişim arayüzü
***************************

.. contents::
   :local:

Bu bölüm kullanıcı ile iletişim kurmamızı sağlayan arayüzlerin kullanımını içerir. 

Mesaj gösterme. QgsMessageBar sınıfı.
=========================================

Mesaj kutularını kullanmak UX (User Experience; Kullanıcı deneyimi) kuralları açısından kötü bir fikirdir. Uyarı ve hata mesajlarını göstermek için bilgi satırını kullanmalıyız. 

QGIS arayüzünde mesaj satırı kısmını kullanarak mesaj göstermek için kullanılacak kod:

.. code-block:: python

  from qgis.core import Qgis
  iface.messageBar().pushMessage("Error", "I'm sorry Dave, I'm afraid I can't do that", level=Qgis.Critical)


.. figure:: img/errorbar.png
   :align: center
   :width: 40em

   QGIS Mesaj çubuğu

Mesaj gösterim süresini ayarlayarak ne kadar süre ile gösterileceğini ayarlayabilirsiniz.

.. code-block:: python

    iface.messageBar().pushMessage("Ooops", "The plugin is not working as it should", level=Qgis.Critical, duration=3)


.. figure:: img/errorbar-timed.png
   :align: center
   :width: 40em

   QGIS Mesaj çubuğu, zamanlanmış

Yukarıdaki örnekte hata mesajı gösterilir, fakat ``level`` parametresi uyarı veya bilgi mesajı tipi :class:`Qgis.MessageLevel <qgis.core.Qgis.MessageLevel>` seçenekleri ile gösterilir. Dört farklı seviye öntanımlıdır:

0. Bilgi
1. Uyarı
2. Kritik
3. Başarılı

.. figure:: img/infobar.png
   :align: center
   :width: 40em

   QGIS Mesaj çubuğu (bilgi)

Bileşenler mesaj çubuğuna eklenebilir. Örneğin daha fazla bilgi göstermek için bir düğme gibi:

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

   QGIS Mesaj çubuğu bir düğme eklenmiş halde

You can even use a message bar in your own dialog so you don't have to show a
message box, or if it doesn't make sense to show it in the main QGIS window

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

   QGIS Message bar in custom dialog


Showing progress
================

Progress bars can also be put in the QGIS message bar, since, as we have seen,
it accepts widgets. Here is an example that you can try in the console.

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

Also, you can use the built-in status bar to report progress, as in the next
example:

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


Logging
=======

You can use the QGIS logging system to log all the information that you want to
save about the execution of your code.

.. code-block:: python

 # You can optionally pass a 'tag' and a 'level' parameters
 QgsMessageLog.logMessage("Your plugin code has been executed correctly", 'MyPlugin', level=Qgis.Info)
 QgsMessageLog.logMessage("Your plugin code might have some problems", level=Qgis.Warning)
 QgsMessageLog.logMessage("Your plugin code has crashed!", level=Qgis.Critical)

.. warning::

 Use of the Python ``print`` statement is unsafe to do in any code which may be
 multithreaded. This includes **expression functions**, **renderers**,
 **symbol layers** and **Processing algorithms** (amongst others). In these
 cases you should always use thread safe classes (:class:`QgsLogger <qgis.core.QgsLogger>`
 or :class:`QgsMessageLog <qgis.core.QgsMessageLog>`) instead.


.. note::

   You can see the output of the :class:`QgsMessageLog <qgis.core.QgsMessageLog>`
   in the :ref:`log_message_panel`

.. note::

 * :class:`QgsLogger <qgis.core.QgsLogger>` is for messages for debugging /
   developers (i.e. you suspect they are triggered by some broken code)
 * :class:`QgsMessageLog <qgis.core.QgsMessageLog>` is for messages to
   investigate issues by sysadmins (e.g. to help a sysadmin to fix configurations)
