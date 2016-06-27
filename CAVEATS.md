## Caveats

There are cases where Qt.py is not handling incompatibility issues.

- [QtCore.QAbstractModel.createIndex](CAVEATS.md#qtcoreqabstractmodelcreateindex)
- [QtCore.QItemSelection](CAVEATS.md#qtcoreqitemselection)
- [QtCore.Slot](CAVEATS.md#qtcoreslot)
- [QtGui.QRegExpValidator](CAVEATS.md#qtguiqregexpvalidator)
- [QtWidgets.QAction.triggered](CAVEATS.md#qtwidgetsqactiontriggered)

<br>
<br>

**Contribute your own caveat**

Pull-requests are welcome, here's how you can make yours.

Code blocks in file are automatically tested on before commited into the project. In order for your code to run successfully, follow these guidelines.

1. Each caveat MUST have a header prefixed with four (4) `#`, e.g. `#### My Heading`.
1. Each caveat SHOULD have example code.
1. Each caveat MAY have one or more example.
1. Each example MAY NOT use more than one (1) binding at a time, e.g. both PyQt5 and PySide.
1. Each example SHOULD `assert` what *is* working, along with what *isn't*.
1. An example MUST reside under a heading, e.g. `#### My Heading`
1. A heading MUST NOT contain anything but letters, numbers, spaces and dots.
1. The first line of each example MUST be `# MyBinding`, where `MyBinding` is the binding you intend to test with, such as `PySide` or `PyQt5`.
1. Examples MUST be in docstring format. See other caveats for samples.
1. Examples MUST `import Qt` (where appropriate), NOT e.g. `import PyQt5`.

<br>
<br>
<br>


#### QtCore.QAbstractItemModel.createIndex

In PySide, somehow the last argument (the id) is allowed to be negative and is maintained. While in PyQt4 it gets coerced into an undefined unsigned value.

```python
# PySide
>>> from Qt import QtGui
>>> model = QtGui.QStandardItemModel()
>>> index = model.createIndex(0, 0, -1)
>>> int(index.internalId())
-1
```

```python
# PyQt4
>>> from Qt import QtGui
>>> model = QtGui.QStandardItemModel()
>>> index = model.createIndex(0, 0, -1)
>>> int(index.internalId())
18446744073709551615
```

> Note - I had been using the id as an index into a list. But the unexpected return value from PyQt4 broke it by being invalid. The workaround was to always check that the returned id was between 0 and the max size I expect.  
– @justinfx

<br>
<br>
<br>

#### QtCore.QItemSelection

PySide has the `QItemSelection.isEmpty` and `QItemSelection.empty` attributes while PyQt4 only has the `QItemSelection.isEmpty` attribute.

```python
# PySide
>>> from Qt import QtCore
>>> assert hasattr(QtCore.QItemSelection, "isEmpty")
>>> assert hasattr(QtCore.QItemSelection, "empty")
```

```python
# PyQt4
>>> from Qt import QtCore
>>> assert hasattr(QtCore.QItemSelection, "isEmpty")
>>> assert not hasattr(QtCore.QItemSelection, "empty")
```

However, they both do support the len(selection) operation.

<br>
<br>
<br>


#### QtCore.Slot

PySide allows for a `result=None` keyword param to set the return type. PyQt4 crashes:

```python
# PySide
>>> from Qt import QtCore, QtGui
>>> assert QtCore.Slot(QtGui.QWidget, result=None)
```

```python
# PyQt4
>>> from Qt import QtCore, QtGui
>>> assert QtCore.Slot(QtGui.QWidget)
>>> assert_raises(TypeError, QtCore.Slot, QtGui.QWidget, result=None)
```


<br>
<br>
<br>

#### QtGui.QRegExpValidator

In PySide, the constructor for `QtGui.QRegExpValidator()` can just take a `QRegExp` instance, and that is all.

In PyQt4 you are required to pass some form of a parent argument, otherwise you get a TypeError:

```python
# PySide
>>> from Qt import QtCore, QtGui
>>> regex = QtCore.QRegExp("\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}")
>>> assert QtGui.QRegExpValidator(regex, None)
>>> assert QtGui.QRegExpValidator(regex)
```

```python
# PyQt4
>>> from Qt import QtCore, QtGui
>>> regex = QtCore.QRegExp("\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}")
>>> assert QtGui.QRegExpValidator(regex, None)
>>> assert_raises(TypeError, QtGui.QRegExpValidator, regex)  # Seems this works fine in PyQt4?
```

<br>
<br>
<br>


#### QtWidgets.QAction.triggered

PySide cannot accept any arguments. In PyQt4, `QAction.triggered` signal requires a bool arg.

```python
# PySide
>>> from Qt import QtCore, QtGui
>>> obj = QtCore.QObject()
>>> action = QtGui.QAction(obj)
>>> assert action.triggered.emit()
>>> assert_raises(TypeError, action.triggered.emit, True)
```

```python
# PyQt4
>>> from Qt import QtCore, QtGui
>>> obj = QtCore.QObject()
>>> action = QtGui.QAction(obj)
>>> try:
...     action.triggered.emit(True)
... except:
...     assert False
... else:
...     assert True
>>> assert_raises(TypeError, action.triggered.emit)
```
