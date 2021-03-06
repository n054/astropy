.. include:: references.txt

.. _construct_table:

Constructing a table
--------------------

There is great deal of flexibility in the way that a table can be initially
constructed.  Details on the inputs to the |Table|
constructor are in the `Initialization Details`_ section.  However, the
easiest way to understand how to make a table is by example.

Examples
^^^^^^^^

Much of the flexibility lies in the types of data structures
which can be used to initialize the table data.  The examples below show how to
create a table from scratch with no initial data, create a table with a list of
columns, a dictionary of columns, or from `numpy` arrays (either structured or
homogeneous).

Setup
"""""
For the following examples you need to import the |Table| and |Column| classes
along with the `numpy` package::

  >>> from astropy.table import Table, Column
  >>> import numpy as np

Creating from scratch
"""""""""""""""""""""
A Table can be created without any initial input data or even without any
initial columns.  This is useful for building tables dynamically if the initial
size, columns, or data are not known.

.. Note::
   Adding columns or rows requires making a new copy of the entire
   table table each time, so in the case of large tables this may be slow.

::

  >>> t = Table()
  >>> t['a'] = [1, 4]
  >>> t['b'] = Column([2.0, 5.0], unit='cm', description='Velocity')
  >>> t['c'] = ['x', 'y']

  >>> t = Table(names=('a', 'b', 'c'), dtype=('f4', 'i4', 'S2'))
  >>> t.add_row((1, 2.0, 'x'))
  >>> t.add_row((4, 5.0, 'y'))


List of columns
"""""""""""""""
A typical case is where you have a number of data columns with the same length
defined in different variables.  These might be Python lists or `numpy` arrays
or a mix of the two.  These can be used to create a |Table| by putting the column
data variables into a Python list.  In this case the column names are not
defined by the input data, so they must either be set using the ``names``
keyword or they will be auto-generated as ``col<N>``.

::

  >>> a = [1, 4]
  >>> b = [2.0, 5.0]
  >>> c = ['x', 'y']
  >>> t = Table([a, b, c], names=('a', 'b', 'c'))
  >>> t
  <Table rows=2 names=('a','b','c')>
  array([(1, 2.0, 'x'), (4, 5.0, 'y')],
        dtype=[('a', '<i8'), ('b', '<f8'), ('c', 'S1')])

**Make a new table using columns from the first table**

Once you have a |Table| then you can make new table by selecting columns
and putting this into a Python list, e.g. ``[ t['c'], t['a'] ]``::

  >>> Table([t['c'], t['a']])
  <Table rows=2 names=('c','a')>
  array([('x', 1), ('y', 4)],
        dtype=[('c', 'S1'), ('a', '<i8')])

**Make a new table using expressions involving columns**

The |Column| object is derived from the standard `numpy` array and can be used
directly in arithmetic expressions.  This allows for a compact way of making a
new table with modified column values::

  >>> Table([t['a']**2, t['b'] + 10])
  <Table rows=2 names=('a','b')>
  array([(1, 12.0), (16, 15.0)],
        dtype=[('a', '<i8'), ('b', '<f8')])


**Different types of column data**

The list input method for |Table| is very flexible since you can use a mix
of different data types to initialize a table::

  >>> a = (1, 4)
  >>> b = np.array([[2, 3], [5, 6]])  # vector column
  >>> c = Column(['x', 'y'], name='axis')
  >>> arr = (a, b, c)
  >>> Table(arr)  # Data column named "c" has a name "axis" that table
  <Table rows=2 names=('col0','col1','axis')>
  array([(1, [2, 3], 'x'), (4, [5, 6], 'y')],
        dtype=[('col0', '<i8'), ('col1', '<i8', (2,)), ('axis', 'S1')])

Notice that in the third column the existing column name ``'axis'`` is used.


Dict of columns
""""""""""""""""
A dictionary of column data can be used to initialize a |Table|.

  >>> arr = {'a': [1, 4],
  ...        'b': [2.0, 5.0],
  ...        'c': ['x', 'y']}
  >>>
  >>> Table(arr)  # doctest: +SKIP
  <Table rows=2 names=('a','c','b')>
  array([(1, 'x', 2.0), (4, 'y', 5.0)],
        dtype=[('a', '<i8'), ('c', 'S1'), ('b', '<f8')])

**Specify the column order and optionally the data types**
::

  >>> Table(arr, names=('a', 'b', 'c'), dtype=('f4', 'i4', 'S2'))
  <Table rows=2 names=('a','b','c')>
  array([(1.0, 2, 'x'), (4.0, 5, 'y')],
        dtype=[('a', '<f4'), ('b', '<i4'), ('c', 'S2')])

**Different types of column data**

The input column data can be any data type that can initialize a |Column| object::

  >>> arr = {'a': (1, 4),
  ...        'b': np.array([[2, 3], [5, 6]]),
  ...        'c': Column(['x', 'y'], name='axis')}
  >>> Table(arr, names=('a', 'b', 'c'))
  <Table rows=2 names=('a','b','c')>
  array([(1, [2, 3], 'x'), (4, [5, 6], 'y')],
        dtype=[('a', '<i8'), ('b', '<i8', (2,)), ('c', 'S1')])

Notice that the key ``'c'`` takes precendence over the existing column name
``'axis'`` in the third column.  Also see that the ``'b'`` column is a vector
column where each row element is itself a 2-element array.

**Renaming columns is not possible**
::

  >>> Table(arr, names=('a_new', 'b_new', 'c_new'))
  Traceback (most recent call last):
    ...
  KeyError: 'a_new'


Row data
"""""""""
Row-oriented data can be used to create a table using the ``rows``
keyword argument.

**List of data records as list or tuple**

If you have row-oriented input data such as a list of records, you
need to use the ``rows`` keyword to create a table::

  >>> data_rows = [(1, 2.0, 'x'),
  ...              (4, 5.0, 'y'),
  ...              (5, 8.2, 'z')]
  >>> t = Table(rows=data_rows, names=('a', 'b', 'c'))
  >>> print(t)
   a   b   c 
  --- --- ---
    1 2.0   x
    4 5.0   y
    5 8.2   z

The data object passed as the ``rows`` argument can be any form which is
parsable by the ``np.rec.fromrecords()`` function.

**List of dict objects**

You can also initialize a table with row values.  This is constructed as a
list of dict objects.  The keys determine the column names::

  >>> data = [{'a': 5, 'b': 10},
  ...         {'a': 15, 'b': 20}]
  >>> Table(rows=data)
  <Table rows=2 names=('a','b')>
  array([(5, 10), (15, 20)],
        dtype=[('a', '<i8'), ('b', '<i8')])

Every row must have the same set of keys or a ValueError will be thrown::

  >>> t = Table(rows=[{'a': 5, 'b': 10}, {'a': 15, 'b': 30, 'c': 50}])
  Traceback (most recent call last):
    ...
  ValueError: Row 0 has no value for column c

**Single row**

You can also make a new table from a single row of an existing table::

  >>> a = [1, 4]
  >>> b = [2.0, 5.0]
  >>> t = Table([a, b], names=('a', 'b'))
  >>> t2 = Table(rows=t[1])

Remember that a |Row| has effectively a zero length compared to the
newly created |Table| which has a length of one.  This is similar to
the difference between a scalar ``1`` (length 0) and an array like
``np.array([1])`` with length 1.

.. Note::

   In the case of input data as a list of dicts or a single Table row, it is
   allowed to supply the data as the ``data`` argument since these forms
   are always unambiguous.  For example ``Table([{'a': 1}, {'a': 2}])`` is
   accepted.  However, a list of records must always be provided using the
   ``rows`` keyword, otherwise it will be interpreted as a list of columns.

NumPy structured array
""""""""""""""""""""""
The structured array is the standard mechanism in `numpy` for storing heterogenous
table data.  Most scientific I/O packages that read table files (e.g.
`PyFITS <http://www.stsci.edu/resources/software_hardware/pyfits>`_,
`vo.table <http://stsdas.stsci.edu/astrolib/vo/html/intro_table.html>`_,
`asciitable <http://cxc.harvard.edu/contrib/asciitable/>`_)
will return the table in an object that is based on the structured array.
A structured array can be created using::

  >>> arr = np.array([(1, 2.0, 'x'),
  ...                 (4, 5.0, 'y')],
  ...                dtype=[('a', 'i8'), ('b', 'f8'), ('c', 'S2')])

From ``arr`` it is simple to create the corresponding |Table| object::

  >>> Table(arr)
  <Table rows=2 names=('a','b','c')>
  array([(1, 2.0, 'x'), (4, 5.0, 'y')],
        dtype=[('a', '<i8'), ('b', '<f8'), ('c', 'S2')])

Note that in the above example and most the following ones we are creating a
table and immediately asking the interactive Python interpreter to print the
table to see what we made.  In real code you might do something like::

  >>> table = Table(arr)
  >>> print table
   a   b   c
  --- --- ---
    1 2.0   x
    4 5.0   y

**New column names**

The column names can be changed from the original values by providing the
``names`` argument::

  >>> Table(arr, names=('a_new', 'b_new', 'c_new'))
  <Table rows=2 names=('a_new','b_new','c_new')>
  array([(1, 2.0, 'x'), (4, 5.0, 'y')],
        dtype=[('a_new', '<i8'), ('b_new', '<f8'), ('c_new', 'S2')])

**New data types**

Likewise the data type for each column can by changed with ``dtype``::

  >>> Table(arr, dtype=('f4', 'i4', 'S4'))
  <Table rows=2 names=('a','b','c')>
  array([(1.0, 2, 'x'), (4.0, 5, 'y')],
        dtype=[('a', '<f4'), ('b', '<i4'), ('c', 'S4')])

  >>> Table(arr, names=('a_new', 'b_new', 'c_new'), dtype=('f4', 'i4', 'S4'))
  <Table rows=2 names=('a_new','b_new','c_new')>
  array([(1.0, 2, 'x'), (4.0, 5, 'y')],
        dtype=[('a_new', '<f4'), ('b_new', '<i4'), ('c_new', 'S4')])



NumPy homogeneous array
"""""""""""""""""""""""
A normal `numpy` 2-d array (where all elements have the same type) can be
converted into a |Table|.  In this case the column names are not specified by
the data and must either be provided by the user or will be automatically
generated as ``col<N>`` where ``<N>`` is the column number.

**Basic example with automatic column names**
::

  >>> arr = np.array([[1, 2, 3],
  ...                 [4, 5, 6]])
  >>> Table(arr)
  <Table rows=2 names=('col0','col1','col2')>
  array([(1, 2, 3), (4, 5, 6)],
        dtype=[('col0', '<i8'), ('col1', '<i8'), ('col2', '<i8')])

**Column names and types specified**
::

  >>> Table(arr, names=('a_new', 'b_new', 'c_new'), dtype=('f4', 'i4', 'S4'))
  <Table rows=2 names=('a_new','b_new','c_new')>
  array([(1.0, 2, '3'), (4.0, 5, '6')],
        dtype=[('a_new', '<f4'), ('b_new', '<i4'), ('c_new', 'S4')])

**Referencing the original data**

It is possible to reference the original data for an homogeneous array as long
as the data types are not changed::

  >>> t = Table(arr, copy=False)

**Python arrays versus `numpy` arrays as input**

There is a slightly subtle issue that is important to understand in the way
that |Table| objects are created.  Any data input that looks like a Python list
(including a tuple) is considered to be a list of columns.  In contrast an
homogeneous `numpy` array input is interpreted as a list of rows::

  >>> arr = [[1, 2, 3],
  ...        [4, 5, 6]]
  >>> np_arr = np.array(arr)

  >>> Table(arr)    # Two columns, three rows
  <Table rows=3 names=('col0','col1')>
  array([(1, 4), (2, 5), (3, 6)],
        dtype=[('col0', '<i8'), ('col1', '<i8')])

  >>> Table(np_arr)  # Three columns, two rows
  <Table rows=2 names=('col0','col1','col2')>
  array([(1, 2, 3), (4, 5, 6)],
        dtype=[('col0', '<i8'), ('col1', '<i8'), ('col2', '<i8')])

This dichotomy is needed to support flexible list input while retaining the
natural interpretation of 2-d `numpy` arrays where the first index corresponds
to data "rows" and the second index corresponds to data "columns".

Table columns
"""""""""""""
A new table can be created by selecting a subset of columns in an existing
table::

  >>> t = Table(names=('a', 'b', 'c'))
  >>> t2 = t['c', 'b', 'a']  # Makes a copy of the data
  >>> print t2
   c   b   a
  --- --- ---

An alternate way to use the ``columns`` attribute (explained in the
`TableColumns`_ section) to initialize a new table.  This let's you choose
columns by their numerical index or name and supports slicing syntax::

  >>> Table(t.columns[0:2])
  <Table rows=0 names=('a','b')>
  array([],
        dtype=[('a', '<f8'), ('b', '<f8')])

  >>> Table([t.columns[0], t.columns['c']])
  <Table rows=0 names=('a','c')>
  array([],
        dtype=[('a', '<f8'), ('c', '<f8')])


Initialization Details
^^^^^^^^^^^^^^^^^^^^^^

A table object is created by initializing a |Table| class
object with the following arguments, all of which are optional:

``data`` : numpy ndarray, dict, list, or Table
    Data to initialize table.
``names`` : list
    Specify column names
``dtype`` : list
    Specify column data types
``meta`` : dict-like
    Meta-Data associated with the table
``copy`` : boolean
    Copy the input data (default=True).

The following subsections provide further detail on the values and options for
each of the keyword arguments that can be used to create a new |Table| object.

data
""""

The |Table| object can be initialized with several different forms
for the ``data`` argument.

**numpy ndarray (structured array)**
    The base column names are the field names of the ``data`` structured
    array.  The ``names`` list (optional) can be used to select
    particular fields and/or reorder the base names.  The ``dtype`` list
    (optional) must match the length of ``names`` and is used to
    override the existing ``data`` types.

**numpy ndarray (homogeneous)**
    The ``data`` ndarray must be at least 2-dimensional, with the first
    (left-most) index corresponding to row number (table length) and the
    second index corresponding to column number (table width).  Higher
    dimensions get absorbed in the shape of each table cell.

    If provided the ``names`` list must match the "width" of the ``data``
    argument.  The default for ``names`` is to auto-generate column names
    in the form "col<N>".  If provided the ``dtype`` list overrides the
    base column types and must match the length of ``names``.

**dict-like**
    The keys of the ``data`` object define the base column names.  The
    corresponding values can be Column objects, numpy arrays, or list-like
    objects.  The ``names`` list (optional) can be used to select
    particular fields and/or reorder the base names.  The ``dtype`` list
    (optional) must match the length of ``names`` and is used to override
    the existing or default data types.

**list-like**
    Each item in the ``data`` list provides a column of data values and
    can be a Column object, numpy array, or list-like object.  The
    ``names`` list defines the name of each column.  The names will be
    auto-generated if not provided (either from the ``names`` argument or
    by Column objects).  If provided the ``names`` argument must match the
    number of items in the ``data`` list.  The optional ``dtype`` list
    will override the existing or default data types and must match
    ``names`` in length.

**list-of-dicts**
    Similar to Python's builtin ``csv.DictReader``, each item in the
    ``data`` list provides a row of data values and must be a dict.  The
    key values in each dict define the column names and each row must
    have identical column names.  The ``names`` argument may be supplied
    to specify colum ordering.  If it is not provided, the column order will
    default to alphabetical.  The ``dtype`` list may be specified, and must
    correspond to the order of output columns.  If any row's keys do no match
    the rest of the rows, a ValueError will be thrown.


**None**
    Initialize a zero-length table.  If ``names`` and optionally ``dtype``
    are provided then the corresponding columns are created.

names
"""""

The ``names`` argument provides a way to specify the table column names or
override the existing ones.  By default the column names are either taken
from existing names (for ``ndarray`` or ``Table`` input) or auto-generated
as ``col<N>``.  If ``names`` is provided then it must be a list with the
same length as the number of columns.  Any list elements with value
``None`` fall back to the default name.

In the case where ``data`` is provided as dict of columns, the ``names``
argument can be supplied to specify the order of columns.  The ``names`` list
must then contain each of the keys in the ``data`` dict.  If ``names`` is not
supplied then the order of columns in the output table is not determinate.

dtype
"""""

The ``dtype`` argument provides a way to specify the table column data
types or override the existing types.  By default the types are either
taken from existing types (for ``ndarray`` or ``Table`` input) or
auto-generated by the ``numpy.array()`` routine.  If ``dtype`` is provided
then it must be a list with the same length as the number of columns.  The
values must be valid ``numpy.dtype`` initializers or ``None``.  Any list
elements with value ``None`` fall back to the default type.

In the case where ``data`` is provided as dict of columns, the ``dtype`` argument
must be accompanied by a corresponding ``names`` argument in order to uniquely
specify the column ordering.

meta
""""

The ``meta`` argument is simply an object that contains meta-data associated
with the table.  It is recommended that this object be a dict or
OrderedDict_, but the only firm requirement is that it can be copied with
the standard library ``copy.deepcopy()`` routine.  By default ``meta`` is
an empty OrderedDict_.

copy
""""

By default the input ``data`` are copied into a new internal ``np.ndarray``
object in the Table object.  In the case where ``data`` is either an
``np.ndarray`` object or an existing ``Table``, it is possible to use a
reference to the existing data by setting ``copy=False``.  This has the
advantage of reducing memory use and being faster.  However one should take
care because any modifications to the new Table data will also be seen in the
original input data.  See the `Copy versus Reference`_ section for more
information.


.. _copy_versus_reference:

Copy versus Reference
^^^^^^^^^^^^^^^^^^^^^

Normally when a new |Table| object is created, the input data are *copied* into
a new internal array object.  This ensures that if the new table elements are
modified then the original data will not be affected.  However, when creating a
table from a numpy ndarray object (structured or homogeneous), it is possible to
disable copying so that instead a memory reference to the original data is
used.  This has the advantage of being faster and using less memory.  However,
caution must be exercised because the new table data and original data will be
linked, as shown below::

  >>> arr = np.array([(1, 2.0, 'x'),
  ...                 (4, 5.0, 'y')],
  ...                dtype=[('a', 'i8'), ('b', 'f8'), ('c', 'S2')])
  >>> print arr['a']  # column "a" of the input array
  [1 4]
  >>> t = Table(arr, copy=False)
  >>> t['a'][1] = 99
  >>> print arr['a']  # arr['a'] got changed when we modified t['a']
  [ 1 99]

Note that when referencing the data it is not possible to change the data types
since that operation requires making a copy of the data.  In this case an error
occurs::

  >>> t = Table(arr, copy=False, dtype=('f4', 'i4', 'S4'))
  Traceback (most recent call last):
    ...
  ValueError: Cannot specify dtype when copy=False

Another caveat in using referenced data is that you cannot add new row to the
table.  This generates an error because of conflict between the two references
to the same underlying memory.  Internally, adding a row may involve moving
the data to a new memory location which would corrupt the input data object.
`numpy` does not allow this::

  >>> t.add_row([1, 2, 3])
  Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
    File "astropy/table/table.py", line 760, in add_row
      self._data.resize((newlen,), refcheck=False)
  ValueError: cannot resize this array: it does not own its data


Column and TableColumns classes
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

There are two classes, |Column| and |TableColumns|, that are useful when
constructing new tables.

Column
""""""

A |Column| object can be created as follows, where in all cases the column
``name`` should be provided as a keyword argument and one can optionally provide
these values:

``data`` : list, ndarray or None
    Column data values
``dtype`` : numpy.dtype compatible value
    Data type for column
``description`` : str
    Full description of column
``unit`` : str
    Physical unit
``format`` : str or function
    `Format specifier`_ for outputting column values
``meta`` : dict
    Meta-data associated with the column

Initialization options
''''''''''''''''''''''

The column data values, shape, and data type are specified in one of two ways:

**Provide a ``data`` value but not a ``length`` or ``shape``**

  Examples::

    col = Column([1, 2], name='a')  # shape=(2,)
    col = Column([[1, 2], [3, 4]], name='a')  # shape=(2, 2)
    col = Column([1, 2], name='a', dtype=float)
    col = Column(np.array([1, 2]), name='a')
    col = Column(['hello', 'world'], name='a')

  The ``dtype`` argument can be any value which is an acceptable
  fixed-size data-type initializer for the numpy.dtype() method.  See
  `<http://docs.scipy.org/doc/numpy/reference/arrays.dtypes.html>`_.
  Examples include:

  - Python non-string type (float, int, bool)
  - Numpy non-string type (e.g. np.float32, np.int64, np.bool)
  - Numpy.dtype array-protocol type strings (e.g. 'i4', 'f8', 'S15')

  If no ``dtype`` value is provided then the type is inferred using
  ``np.array(data)``.  When ``data`` is provided then the ``shape``
  and ``length`` arguments are ignored.

**Provide ``length`` and optionally ``shape``, but not ``data``**

  Examples::

    col = Column(name='a', length=5)
    col = Column(name='a', dtype=int, length=10, shape=(3,4))

  The default ``dtype`` is ``np.float64``.  The ``shape`` argument is the array shape of a
  single cell in the column.  The default ``shape`` is () which means a single value in
  each element.

.. note::

   After setting the type for a column, that type cannot be changed.
   If data values of a different type are assigned to the column then they
   will be cast to the existing column type.

.. _table_format_string:

Format specifier
''''''''''''''''

The format specifier controls the output of column values when a table or column
is printed or written to an ASCII table.  The format specifier can be either
a "old-style" or "new-style" format string or a function:

**Old-style**

This corresponds to syntax like ``"%.4f" % value`` as documented in
`String formatting operations <http://docs.python.org/library/stdtypes.html#string-formatting-operations>`_.

   ``"%.4f"`` to print four digits after the decimal in float format, or

   ``"%6d"`` to print an integer in a 6-character wide field.

**New-style**

This corresponds to syntax like ``"{:.4f}".format(value)`` as documented in
`format string syntax
<http://docs.python.org/library/string.html#format-string-syntax>`_.

   ``"{:.4f}"`` to print four digits after the decimal in float format, or

   ``"{:6d}"`` to print an integer in a 6-character wide field.

Note that in either case any Python format string that formats exactly
one value is valid, so ``{:.4f} angstroms`` or ``Value: %12.2f`` would both work.

**Function**

The greatest flexibility can be achieved by setting a formatting function. This
function must accept a single argument (the value) and return a string. In the
following example this is used to make a LaTeX ready output::

    >>> t = Table([[1,2],[1.234e9,2.34e-12]], names = ('a','b'))
    >>> def latex_exp(value):
    ...     val = '{0:8.2}'.format(value)
    ...     mant, exp = val.split('e')
    ...     # remove leading zeros
    ...     exp = exp[0] + exp[1:].lstrip('0')
    ...     return '$ {0} \\times 10^{{ {1} }}$' .format(mant, exp)
    >>> t['b'].format = latex_exp
    >>> t['a'].format = '{0:.4f}'
    >>> import sys
    >>> t.write(sys.stdout, format='latex')
    \begin{table}
    \begin{tabular}{cc}
    a & b \\
    1.0000 & $  1.2 \times 10^{ +9 }$ \\
    2.0000 & $  2.3 \times 10^{ -12 }$ \\
    \end{tabular}
    \end{table}


TableColumns
""""""""""""

Each |Table| object has an attribute ``columns`` which is an ordered dictionary
that stores all of the |Column| objects in the table (see also the `Column`_
section).  Technically the ``columns`` attribute is a |TableColumns| object,
which is an enhanced ordered dictionary that provides easier ways to select
multiple columns.  There are a few key points to remember:

- A |Table| can be initialized from a |TableColumns| object (copy is always True).
- Selecting multiple columns from a |TableColumns| object returns another
  |TableColumns| object.
- Select one column from a |TableColumns| object returns a |Column|.

So now look at the ways to select columns from a |TableColumns| object:

**Select columns by name**
::

  >>> t = Table(names=('a', 'b', 'c', 'd'))

  >>> t.columns['d', 'c', 'b']
  <TableColumns names=('d','c','b')>

**Select columns by index slicing**
::

  >>> t.columns[0:2]  # Select first two columns
  <TableColumns names=('a','b')>

  >>> t.columns[::-1]  # Reverse column order
  <TableColumns names=('d','c','b','a')>

**Select column by index or name**
::

  >>> t.columns[1]  # Choose columns by index
  <Column name='b' unit=None format=None description=None>
  array([], dtype=float64)

  >>> t.columns['b']  # Choose column by name
  <Column name='b' unit=None format=None description=None>
  array([], dtype=float64)


.. _subclassing_table:

Subclassing Table
^^^^^^^^^^^^^^^^^

For some applications it can be useful to subclass the |Table| class in order
to introduce specialized behavior.  In addition to subclassing |Table| it is
frequently desirable to change the behavior of the internal class objects which
are contained or created by a Table.  This includes rows, columns, formatting,
and the columns container.  In order to do this the subclass needs to declare
what class to use (if it is different from the built-in version).  This is done by
specifying one or more of the class attributes ``Row``, ``Column``,
``MaskedColumn``, ``TableColumns``, or ``TableFormatter``.

The following trivial example overrides all of these with do-nothing
subclasses, but in practice you would override only the necessary subcomponents::

  >>> from astropy.table import Table, Row, Column, MaskedColumn, TableColumns, TableFormatter

  >>> class MyRow(Row): pass
  >>> class MyColumn(Column): pass
  >>> class MyMaskedColumn(MaskedColumn): pass
  >>> class MyTableColumns(TableColumns): pass
  >>> class MyTableFormatter(TableFormatter): pass

  >>> class MyTable(Table):
  ...     """
  ...     Custom subclass of astropy.table.Table
  ...     """
  ...     Row = MyRow  # Use MyRow to create a row object
  ...     Column = MyColumn  # Column
  ...     MaskedColumn = MyMaskedColumn  # Masked Column
  ...     TableColumns = MyTableColumns  # Ordered dict holding Column objects
  ...     TableFormatter = MyTableFormatter  # Controls table output


Example
""""""""

As a more practical example, suppose you have a table of data with a certain set of fixed
columns, but you also want to carry an arbitrary dictionary of keyword=value
parameters for each row and then access those values using the same item access
syntax as if they were columns.  It is assumed here that the extra parameters
are contained in a numpy object-dtype column named ``params``::

  >>> from astropy.table import Table, Row
  >>> class ParamsRow(Row):
  ...    """
  ...    Row class that allows access to an arbitrary dict of parameters
  ...    stored as a dict object in the ``params`` column.
  ...    """
  ...    def __getitem__(self, item):
  ...        if item not in self.colnames:
  ...            return self.data['params'][item]
  ...        else:
  ...            return self.data[item]
  ...
  ...    def keys(self):
  ...        out = [name for name in self.colnames if name != 'params']
  ...        params = [key.lower() for key in sorted(self.data['params'])]
  ...        return out + params
  ...
  ...    def values(self):
  ...        return [self[key] for key in self.keys()]

Now we put this into action with a trival |Table| subclass::

  >>> class ParamsTable(Table):
  ...     Row = ParamsRow

First make a table and add a couple of rows::

  >>> t = ParamsTable(names=['a', 'b', 'params'], dtype=['i', 'f', 'O'])
  >>> t.add_row((1, 2.0, {'x': 1.5, 'y': 2.5}))
  >>> t.add_row((2, 3.0, {'z': 'hello', 'id': 123123}))
  >>> print(t)  # doctest: +SKIP
   a   b             params
  --- --- ----------------------------
    1 2.0         {'y': 2.5, 'x': 1.5}
    2 3.0 {'z': 'hello', 'id': 123123}

Now see what we have from our specialized ``ParamsRow`` object::

  >>> t[0]['y']
  2.5
  >>> t[1]['id']
  123123
  >>> t[1].keys()
  ['a', 'b', 'id', 'z']
  >>> t[1].values()
  [2, 3.0, 123123, 'hello']

To make this example really useful you might want to override
``Table.__getitem__`` in order to allow table-level access to the parameter
fields.  This might look something like::

  class ParamsTable(table.Table):
      Row = ParamsRow

      def __getitem__(self, item):
          if isinstance(item, six.string_types):
              if item in self.colnames:
                  return self.columns[item]
              else:
                  # If item is not a column name then create a new MaskedArray
                  # corresponding to self['params'][item] for each row.  This
                  # might not exist in some rows so mark as masked (missing) in
                  # those cases.
                  mask = np.zeros(len(self), dtype=np.bool)
                  item = item.upper()
                  values = [params.get(item) for params in self['params']]
                  for ii, value in enumerate(values):
                      if value is None:
                          mask[ii] = True
                          values[ii] = ''
                  return self.MaskedColumn(name=item, data=values, mask=mask)

          # ... and then the rest of the original __getitem__ ...
