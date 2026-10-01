# FST Indicator GUI BUG
when right clicking on the status indicator to open the context menu. no menu opens and Error raised.

Error calling Python override of QWidget::contextMenuEvent(): Traceback (most recent call last):
  File "C:\Users\Wasiejen\Projects\OpenCodeProjects\Free-Snap-Tap\Free-Snap-Tap\fst_overlay.py", line 595, in contextMenuEvent
    self.context_menu.exec_(event.globalPosition().toPoint())
                            ^^^^^^^^^^^^^^^^^^^^
AttributeError: 'PySide6.QtGui.QContextMenuEvent' object has no attribute 'globalPosition'

# FST Toastbox bug
when first time triggering a function that is sending a toast message to be displayed under the status indicator the input is lagging extremely for around 2 seconds (mouse inclusive) until the toast message is displayed. (that it is cause by the toast message is a guess because this is the only observable change for me right now)
- not such delay on retriggering the same macro with the toast message appearing immediately
