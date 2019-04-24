Using the OGM
=============

:py:mod:`Djraph<djraph>` aims to provide a powerful Object Graph Mapper
**(OGM)** while maintaining a simple, transparent interface. This document
describes the OGM components in more detail.

Modeling Graph Elements with :py:mod:`Djraph<djraph>`
-----------------------------------------------------

At the core of the :py:mod:`Djraph<djraph>` is the concept of the graph
element. TinkerPop 3 (TP3) uses three basic kinds of elements: Vertex, Edge,
and Property. In order to achieve consistent mapping between Python objects and
TP3 elements, :py:mod:`Djraph<djraph>` provides three corresponding Python base
classes that are used to model graph data:
:py:class:`Vertex<djraph.element.Vertex>`,
:py:class:`Edge<djraph.element.Edge>`, and
:py:class:`Property<djraph.properties.Property>`. While these classes are
created to interact smoothly with TP3, it is important to remember that
:py:mod:`Djraph<djraph>` does not attempt to implement the same element
interface found in TP3. Indeed, other than user defined properties,
:py:mod:`Djraph<djraph>` elements feature little to no interface. To begin
modeling data, simply create *model* element classes that inherit from the
:py:mod:`djraph.element` classes. For example::

    import djraph
    from gremlin_python import Cardinality


    class Person(djraph.Vertex):
        pass


    class City(djraph.Vertex):
        pass


    class BornIn(djraph.Edge):
        pass


And that is it, these are valid element classes that can be saved to the graph
database. Using these three classes we can model a series of people that are connected
to the cities in which they were born. However, these elements
aren't very useful, as they don't contain any information about the person or place. To remedy
this, add some properties to the classes.

Using :py:mod:`djraph.properties`
---------------------------------

Using the :py:mod:`properties<djraph.properties>` module is a bit more
involved, but it is still pretty easy. It simply requires that you create
properties that are defined as Python class attributes, and each property
requires that you pass a :py:class:`DataType<djraph.abc.DataType>` class **or**
instance as the first positional argument. This data type, which is a concrete
class that inherits from :py:class:`DataType<djraph.abc.DataType>`, handles
validation, as well as any necessary conversion when data is mapped between the
database and the OGM. :py:mod:`Djraph<djraph>` currently ships with 4 data
types: :py:class:`String<djraph.properties.String>`,
:py:class:`Integer<djraph.properties.Integer>`,
:py:class:`Float<djraph.properties.Float>`, and
:py:class:`Boolean<djraph.properties.Boolean>`. Example property definition::


    >>> import djraph
    >>> class Person(djraph.Vertex):
    ...    name = djraph.Property(djraph.String)
    >>> class City(djraph.Vertex):
    ...     name = djraph.Property(djraph.String)
    ...     population = djraph.Property(djraph.Integer)
    >>> class BornIn(djraph.Edge):
    ...     pass


:py:mod:`Djraph<djraph>` :py:mod:`properties<djraph.properties.Property>` can
also be created with a default value, set by using the kwarg `default` in the
class definition::


    >>> class BornIn(djraph.Edge):
    ...    date = djraph.Property(djraph.String, default='unknown')


Creating Elements and Setting Property Values
---------------------------------------------

Behind the scenes, a small metaclass (the only metaclass used in
:py:mod:`Djraph<djraph>`), substitutes
a :py:class:`PropertyDescriptor<djraph.properties.PropertyDescriptor>` for the
:py:class:`Property<djraph.properties.Property>`, which provides a simple
interface for defining and updating properties using Python's descriptor
protocol::

    >>> leif = Person()
    >>> leif.name = 'Leif'

    >>> detroit = City()
    >>> detroit.name = 'Detroit'
    >>> detroit.population = 	5311449  # CSA population

    # change a property value
    >>> leif.name = 'Leifur'

In the case that an invalid property value is set, the validator will raise
a :py:class:`ValidationError<djraph.exception.ValidationError>` immediately::


    >>> detroit.population = 'a lot of people'
    Traceback (most recent call last):
      ...
    djraph.exception.ValidationError: Not a valid integer: a lot of people


Creating Edges -------------- Creating edges is very similar to creating
vertices, except that edges require that a source (outV) and target (inV)
vertex be specified. Both source and target nodes must be :py:mod:`Djraph
vertices<djraph.element.Vertex>`. Furthermore, they must be created in the
database before the edge. This is further discussed below in the
:ref:`Session<session>` section. Source and target vertices may be passed to
the edge on instantiation, or added using the property interface::

    >>> leif_born_in_detroit = BornIn(leif, detroit)
    >>> # or
    >>> leif_born_in_detroit = BornIn()
    >>> leif_born_in_detroit.source = leif
    >>> leif_born_in_detroit.target = detroit
    >>> leif_born_in_detroit.date  # default value
    'unknown'


Vertex Properties
-----------------

In addition to the aforementioned elements, TP3 graphs also use a special kind
of property, called a vertex property, that allows for list/set cardinality and
meta-properties. To accommodate this, :py:mod:`Djraph<djraph>` provides a class
:py:class:`VertexProperty<djraph.element.VertexProperty>` that can be used
directly to create multi-cardinality properties::

    >>> from gremlin_python.process.traversal import Cardinality
    >>> class Person(djraph.Vertex):
    ...     name = djraph.Property(djraph.String)
    ...     nicknames = djraph.VertexProperty(
    ...         djraph.String, card=Cardinality.list_)


    >>> david = Person()
    >>> david.name = 'David'
    >>> david.nicknames = ['Dave', 'davebshow']


Notice that the cardinality of the
:py:class:`VertexProperty<djraph.element.VertexProperty>` must be explicitly
set using the `card` kwarg and the
:py:class:`Cardinality<gremlin_python.process.traversal.Cardinality>`
enumerator.

.. autoclass:: gremlin_python.process.traversal.Cardinality
   :members:
   :undoc-members:

:py:class:`VertexProperty<djraph.element.VertexProperty>` provides a different
interface than the simple, key/value style
:py:class:`PropertyDescriptor<djraph.properties.PropertyDescriptor>` in order
to accomodate more advanced functionality. For accessing multi-cardinality
vertex properties, :py:mod:`Djraph<djraph>` provides several helper classes
called :py:mod:`managers<djraph.manager>`. The
:py:class:`managers<djraph.manager.ListVertexPropertyManager>` inherits from
:py:class:`list` or :py:class:`set` (depending on the specified cardinality),
and provide a simple API for accessing and appending vertex properties. To
continue with the previous example, we see the `dave` element's nicknames::

    >>> david.nicknames
    [<VertexProperty(type=<...>, value=Dave), <VertexProperty(type=<...>, value=davebshow)]

To add a nickname without replacing the earlier values, we simple
:py:meth:`append<djraph.manager.ListVertexPropertyManager.append>` as if the
manager were a Python :py:class:`list`::

    >>> david.nicknames.append('db')
    >>> david.nicknames
    [<VertexProperty(type=<...>, value=Dave), <VertexProperty(type=<...>, value=davebshow), <VertexProperty(type=<...>, value=db)]

If this were a :py:class:`VertexProperty<djraph.element.VertexProperty>` with
a set cardinality, we would simply use
:py:meth:`add<djraph.manager.SetVertexPropertyManager.add>` to achieve similar
functionality.

Both :py:class:`ListVertexPropertyManager<djraph.manager.ListVertexPropertyManager>` and
:py:class:`SetVertexPropertyManager<djraph.manager.SetVertexPropertyManager>` provide a simple
way to access a specific :py:class:`VertexProperty<djraph.element.VertexProperty>`.
You simply call the manager, passing the value of the vertex property to be accessed:

    >>> db = david.nicknames('db')

The value of the vertex property can be accessed using the `value` property::

    >>> db.value
    'db'


Meta-properties
---------------

:py:class:`VertexProperty<djraph.element.VertexProperty>` can also be used as
a base classes for user defined vertex properties that contain meta-properties.
To create meta-properties, define a custom vertex property class just like you
would any other element, adding as many simple (non-vertex) properties as needed::

    >>> class HistoricalName(djraph.VertexProperty):
    ...     notes = djraph.Property(djraph.String)

Now, the custom :py:class:`VertexProperty<djraph.element.VertexProperty>` can be added to a
vertex class, using any cardinality::

    >>> class City(djraph.Vertex):
    ...     name = djraph.Property(djraph.String)
    ...     population = djraph.Property(djraph.Integer)
    ...     historical_name = HistoricalName(
    ...         djraph.String, card=Cardinality.list_)

Now, meta-properties can be set on the :py:class:`VertexProperty<djraph.element.VertexProperty>`
using the descriptor protocol::

    >>> montreal = City()
    >>> montreal.historical_name = ['Ville-Marie']
    >>> montreal.historical_name('Ville-Marie').notes = 'Changed in 1705'

And that's it.

.. _session:

Saving Elements to the Database Using :py:class:`Session<djraph.session.Session>`
---------------------------------------------------------------------------------

All interaction with the database is achieved using the
:py:class:`Session<djraph.session.Session>` object. A :py:mod:`Djraph<djraph>`
session should not be confused with a Gremlin Server session, although in
future releases it will provide support for server sessions and transactions.
Instead, the :py:class:`Session<djraph.session.Session>` object is used to save
elements and spawn Gremlin traversals. Furthemore, any element created using
a session is *live* in the sense that
a :py:class:`Session<djraph.session.Session>` object maintains a reference to
session elements, and if a traversal executed using a session returns different
property values for a session element, these values are automatically updated
on the session element. Note - the examples shown in this section must be
wrapped in coroutines and ran using the :py:class:`asyncio.BaseEventLoop`, but,
for convenience, they are shown as if they were run in a Python interpreter. To
use a :py:class:`Session<djraph.session.Session>`, first create
a :py:class:`Djraph App <djraph.app.Djraph>` using
:py:meth:`Djraph.open<djraph.app.Djraph.open>`,

.. code-block:: python

    app = await djraph.Djraph.open(loop)


then register the defined element classes::

    >>> app.register(Person, City, BornIn)

Djraph application support a variety of configuration options, for more
information see :doc:`the Djraph application documentation</app>`.

The best way to create elements is by adding them to the session, and then flushing
the `pending` queue, thereby creating the elements in the database. The order in which
elements are added **is** important, as elements will be created based on the order
in which they are added. Therefore, when creating edges, it is important to add the
source and target nodes before the edge (if they don't already exits). Using
the previously created elements::

    >>> async def create_elements(app):
    ...     session = await app.session()
    ...     session.add(leif, detroit, leif_born_in_detroit)
    ...     await session.flush()
    >>> loop.run_until_complete(create_elements(app))


And that is it. To see that these elements have actually been created in the db,
check that they now have unique ids assigned to them::

    >>> assert leif.id
    >>> assert detroit.id
    >>> assert leif_born_in_detroit.id

For more information on the :py:class:`Djraph App <djraph.app.Djraph>`, please
see :doc:`Using the Djraph App</app>`

:py:class:`Session<djraph.session.Session>` provides a variety of other CRUD
functions, but all creation and updating can be achieved simply using the
:py:meth:`add<djraph.session.Session.add>` and
:py:meth:`flush<djraph.session.Session.flush>` methods.


Writing Custom Gremlin Traversals
---------------------------------

Finally, :py:class:`Session<djraph.session.Session>` objects allow you to write
custom Gremlin traversals using the official gremlin-python Gremlin Language
Variant **(GLV)**. There are two methods available for writing session based
traversals. The first, :py:meth:`traversal<djraph.session.Session.traversal>`,
accepts an element class as a positional argument. This is merely for
convenience, and generates this equivalent Gremlin::

    >>> session = loop.run_until_complete(app.session())
    >>> session.traversal(Person)
    [['V'], ['hasLabel', 'person']]

Or, simply use the property :py:attr:`g<djraph.session.Session.g>`::

    >>> session.g.V().hasLabel('person')
    [['V'], ['hasLabel', 'person']]


In general property names are mapped directly from the OGM to the database.
However, by passing the `db_name` kwarg to a property definition, the user has
the ability to override this behavior. To avoid mistakes in the case of custom
database property names, it is encouraged to access the mapped property names
as class attributes::

    >>> Person.name
    'name'

So, to write a traversal::

    >>> session.traversal(Person).has(Person.name, 'Leifur')
    [['V'], ['hasLabel', 'person'], ['has', 'name', 'Leifur']]


Also, it is important to note that certain data types could be transformed
before they are written to the database. Therefore, the data type method
`to_db` may be required::

    >>> session.traversal(Person).has(
    ...     Person.name, djraph.String().to_db('Leifur'))
    [['V'], ['hasLabel', 'person'], ['has', 'name', 'Leifur']]

While this is not the case with any of the simple data types shipped with
:py:mod:`Djraph<djraph>`, custom data types or future additions may require
this kind of operation. Because of this, :py:mod:`Djraph<djraph>` includes the
convenience function :py:func:`bindprop<djraph.session.bindprop>`, which also
allows an optional binding for the value to be specified::

    >>> from djraph.session import bindprop
    >>> traversal = session.traversal(Person)
    >>> traversal.has(bindprop(Person, 'name', 'Leifur', binding='v1'))
    [['V'], ['hasLabel', 'person'], ['has', binding[name=binding[v1=Leifur]]]]

Finally, there are a variety of ways to to submit a traversal to the server.
First of all, all traversals are themselve asynchronous iterators, and using
them as such will cause a traversal to be sent on the wire:

.. code-block:: python

    async for msg in session.g.V().hasLabel('person'):
         print(msg)

Furthermore, :py:mod:`Djraph<djraph>` provides several convenience methods that
submit a traversal as well as process the results
:py:meth:`toList<aiogremlin.process.graph_traversal.AsyncGraphTraversal.toList>`,
:py:meth:`toSet<aiogremlin.process.graph_traversal.AsyncGraphTraversal.toSet>`
and :py:meth:`next<aiogremlin.process.graph_traversal.AsyncGraphTraversal.next>`.
These methods both submit a script to the server and iterate over the results.
Remember to `await` the traversal when calling these methods:

.. code-block:: python

    traversal = session.traversal(Person)
    leif = await traversal.has(
        bindprop(Person, 'name', 'Leifur', binding='v1')).next()

And that is pretty much it. We hope you enjoy the :py:mod:`Djraph<djraph>` OGM.
