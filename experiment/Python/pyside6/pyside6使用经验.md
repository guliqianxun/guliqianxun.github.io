pyside6使用经验

```python
import sys
import random
from PySide6 import QtCore, QtWidgets, QtGui

class MyWidget(QtWidgets.QWidget):
    def __init__(self):
        super().__init__()

        self.hello = ["Hallo Welt", "Hei maailma", "Hola Mundo", "Привет мир"]

        self.button = QtWidgets.QPushButton("Click me!")
        self.text = QtWidgets.QLabel("Hello World",
                                     alignment=QtCore.Qt.AlignCenter)

        self.layout = QtWidgets.QVBoxLayout()
        self.layout.addWidget(self.text)
        self.layout.addWidget(self.button)
        self.setLayout(self.layout)

        self.button.clicked.connect(self.magic)

    @QtCore.Slot()
    def magic(self):
        self.text.setText(random.choice(self.hello))

if __name__ == "__main__":
    app = QtWidgets.QApplication([])

    widget = MyWidget()
    widget.resize(800, 600)
    widget.show()

    sys.exit(app.exec_())
```

##### dll引用错误

按照官网给的例程，发现了如下错误：

```
Found invalid metadata in lib D:/soft/anaconda3/Library/plugins/platforms/qdirect2d.dll: Invalid metadata version
Found invalid metadata in lib D:/soft/anaconda3/Library/plugins/platforms/qminimal.dll: Invalid metadata version
Found invalid metadata in lib D:/soft/anaconda3/Library/plugins/platforms/qoffscreen.dll: Invalid metadata version
Found invalid metadata in lib D:/soft/anaconda3/Library/plugins/platforms/qwindows.dll: Invalid metadata version
qt.qpa.plugin: Could not find the Qt platform plugin "windows" in ""
This application failed to start because no Qt platform plugin could be initialized. Reinstalling the application may fix this problem.
```

这个错误第一眼很容易让人误认为是环境变量的原因，特别是我在使用anconda的虚拟环境，就认为是环境变量没添加对，浪费了一点时间。

实际上是因为我在之前就按照了pyqt5，然后pyside在引用库时，引用了pyqt5的dll。

###### **解决方案：**

把pyside的库复制出来

```py
\Anaconda3\Lib\site-packages\PySide2\plugins\platforms\qminimal.dll
\Anaconda3\Lib\site-packages\PySide2\plugins\platforms\qoffscreen.dll
\Anaconda3\Lib\site-packages\PySide2\plugins\platforms\qwindows.dll
```

替换library里的库

```py
\Anaconda3\Library\plugins\platforms\
```