Section 1
---------
.. code:: python
    
    from datetime import datetime
    print(datetime.now())

.. note::
   Careful! Don't go overtime!

.. danger::
    This step is dangerous! Be real sure about it!

.. Seealso::
   https://docutils.sourceforge.io/docs/ref/rst/directives.html#specific-admonitions

========  ========  ========
Column A  Column B  Column C
========  ========  ========
row 1A    row 1B    row 1C
row 2A    row 2B    row 2C
========  ========  ========

+----------+----------+----------+
| Column A | Column B | Column C |
+==========+==========+==========+
| row 1A   | row 1B   | row 1C   |
+----------+----------+----------+
| row 2A   | row 2B   | row 2C   |
+----------+----------+----------+


.. csv-table::
   :header: "Column A", "Column B", "Column C"

   "row 1A", "row 1B", "row 1C"
   "row 2A", "row 2B", "row 2C"


------------------
Subsection 1.1.1.1
------------------
Section 2
---------
.. image:: path_to_image.jpg

SubSection 2.1
==============

SubSection 2.1.1
++++++++++++++++

