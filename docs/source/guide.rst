Developer Guide
===============

Before Starting
---------------

Before using NoMETA as a developer, you should be familiar with the Office Open XML (OOXML) properties defined in `ECMA-376 <https://ecma-international.org/publications-and-standards/standards/ecma-376/>`_.
So, you can understand the Microsoft implementation of *app.xml* and *core.xml* in *docProps* folder structure.

Introduction
------------

The main class of NoMETA is :class:`Document <nometa.Document>`, it's responsible for opening and saving MS Office documents. It also agregates two classes, :class:`App <nometa.sheet.App>` and :class:`Core <nometa.sheet.Core>`, to deal with XML tag values.

In context of NoMETA, ``App`` and ``Core`` are sheets that represents the XML files *docProps/app.xml* and *docProps/core.xml*, respectively. These classes have properties to deal with the most popular tags of each file.
By convention, each property has the same name of the corresponding XML tag in **snake_case** format (eg. ``<cp:lastModifiedBy>`` is :attr:`last_modified_by <nometa.sheet.Core.last_modified_by>`)

So, you can edit these properties and save a new version of the document like in the code snippet below:

.. code-block:: python
    :linenos:

    from nometa import Document, Core, App
    doc = Document("tests/resource/test.docx",Core,App)
    doc.core.creator = "Johnny Test"
    doc.save("test2.docx")

|

.. tip::

    The constructor of Document class can accept, as first parameter, a string (which is the file path) or an in-memory buffer of bytes.
    The ``save`` method behaves similarly.
    
    See the class documentation for more details.

|

How to handle a new XML tag?
----------------------------

If ``App`` or ``Core`` classes don't handle some XML tag, you can extend them in order to handle the specific tag. As an example, suppose that a new tag called **dc:unreal** is present in *docProps/core.xml*, so you could access it as following:

.. code-block:: python
    :linenos:
    :emphasize-lines: 8,12

    from nometa import *
    from typing import cast
    from nometa.properties.text import TextProperty

    class UnrealCore(Core):
        def __init__(self, raw) -> None:
            super().__init__(raw)
            self._unreal=TextProperty.from_element("dc:unreal", self._xml_root)

        def to_element(self) -> None:
            super().to_element()
            self._unreal.to_element(self._xml_root)

        @property
        def unreal(self) -> str|None:
            return self._unreal.value
        
        @unreal.setter
        def unreal(self, val: str) -> None:
            self._unreal.value=val
    
    
    doc=Document("tests/resource/test.xlsx",UnrealCore,App)
    ucore=cast(UnrealCore, doc.core)
    print(ucore.unreal)

The attribute ``_xml_root`` of :class:`nometa.sheet.Sheet` class keeps an instance of ``etree._Element`` class, and you must *read from* and *write to* it. Like above in lines 8 and 12.