# App Programming Quickest-start

This shows you how to add code into a file, add it to main menu and run it.
See https://github.com/emfcamp/tidal-docs/blob/main/AppQuickstart.md for more example code!

1. Plug in your TiDAL in your computer with power off
2. Turn on your TiDAL
3. Open https://editor.badge.emfcamp.org/ in Google Chrome
4. Press "Connect", select your TiDAL (Espressif CDC Device on Windows) and press "Connect"
5. Press "Programming & Files"
6. Press "Make folder", second button from the left, and create a folder named `apps`
7. Create a folder for your app in `apps`, e.g. `helloworld`
8. Select your app folder and press "Make file", call this `__init__.py`
9. Select `__init__.py` - the filename/path should be `/apps/helloworld/__init__.py`
10. Paste the following code into the file:
    ```Python
    from tidal import *
    from textwindow import TextWindow

    win = TextWindow(bg=BLACK, fg=WHITE)
    win.cls()
    win.println("Hello world!")
    ```
11. Press "Save file", the rightmost button
12. Reset your TiDAL by pressing the button in the corner and see your app in the list on your TiDAL
