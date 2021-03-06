.. index::
   single: Forms; Fields; EntityType

EntityType Field
================

A special ``ChoiceType`` field that's designed to load options from a Doctrine
entity. For example, if you have a ``Category`` entity, you could use this
field to display a ``select`` field of all, or some, of the ``Category``
objects from the database.

+-------------+------------------------------------------------------------------+
| Rendered as | can be various tags (see :ref:`forms-reference-choice-tags`)     |
+-------------+------------------------------------------------------------------+
| Options     | - `choice_label`_                                                |
|             | - `class`_                                                       |
|             | - `em`_                                                          |
|             | - `query_builder`_                                               |
+-------------+------------------------------------------------------------------+
| Overridden  | - `choice_name`_                                                 |
| options     | - `choice_value`_                                                |
|             | - `choices`_                                                     |
|             | - `data_class`_                                                  |
+-------------+------------------------------------------------------------------+
| Inherited   | from the :doc:`ChoiceType </reference/forms/types/choice>`:      |
| options     |                                                                  |
|             | - `choice_attr`_                                                 |
|             | - `choice_translation_domain`_                                   |
|             | - `expanded`_                                                    |
|             | - `group_by`_                                                    |
|             | - `multiple`_                                                    |
|             | - `placeholder`_                                                 |
|             | - `preferred_choices`_                                           |
|             | - `translation_domain`_                                          |
|             |                                                                  |
|             | from the :doc:`FormType </reference/forms/types/form>`:          |
|             |                                                                  |
|             | - `data`_                                                        |
|             | - `disabled`_                                                    |
|             | - `empty_data`_                                                  |
|             | - `error_bubbling`_                                              |
|             | - `error_mapping`_                                               |
|             | - `label`_                                                       |
|             | - `label_attr`_                                                  |
|             | - `label_format`_                                                |
|             | - `mapped`_                                                      |
|             | - `required`_                                                    |
+-------------+------------------------------------------------------------------+
| Parent type | :doc:`ChoiceType </reference/forms/types/choice>`                |
+-------------+------------------------------------------------------------------+
| Class       | :class:`Symfony\\Bridge\\Doctrine\\Form\\Type\\EntityType`       |
+-------------+------------------------------------------------------------------+

Basic Usage
-----------

The ``entity`` type has just one required option: the entity which should
be listed inside the choice field::

    use Symfony\Bridge\Doctrine\Form\Type\EntityType;
    // ...

    $builder->add('users', EntityType::class, array(
        // query choices from this entity
        'class' => 'AppBundle:User',

        // use the User.username property as the visible option string
        'choice_label' => 'username',

        // used to render a select box, check boxes or radios
        // 'multiple' => true,
        // 'expanded' => true,
    ));

This will build a ``select`` drop-down containing *all* of the ``User`` objects
in the database. To render radio buttons or checkboxes instead, change the
`multiple`_ and `expanded`_ options.

.. _ref-form-entity-query-builder:

Using a Custom Query for the Entities
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you want to create a custom query to use when fetching the entities
(e.g. you only want to return some entities, or need to order them), use
the `query_builder`_ option::

    use Doctrine\ORM\EntityRepository;
    use Symfony\Bridge\Doctrine\Form\Type\EntityType;
    // ...

    $builder->add('users', EntityType::class, array(
        'class' => 'AppBundle:User',
        'query_builder' => function (EntityRepository $er) {
            return $er->createQueryBuilder('u')
                ->orderBy('u.username', 'ASC');
        },
        'choice_label' => 'username',
    ));

.. _reference-forms-entity-choices:

Using Choices
~~~~~~~~~~~~~

If you already have the exact collection of entities that you want to include
in the choice element, just pass them via the ``choices`` key.

For example, if you have a ``$group`` variable (passed into your form perhaps
as a form option) and ``getUsers`` returns a collection of ``User`` entities,
then you can supply the ``choices`` option directly::

    use Symfony\Bridge\Doctrine\Form\Type\EntityType;
    // ...

    $builder->add('users', EntityType::class, array(
        'class' => 'AppBundle:User',
        'choices' => $group->getUsers(),
    ));

.. include:: /reference/forms/types/options/select_how_rendered.rst.inc

Field Options
-------------

choice_label
~~~~~~~~~~~~

**type**: ``string``, ``callable`` or :class:`Symfony\\Component\\PropertyAccess\\PropertyPath`

This is the property that should be used for displaying the entities as text in
the HTML element::

    use Symfony\Bridge\Doctrine\Form\Type\EntityType;
    // ...

    $builder->add('category', EntityType::class, array(
        'class' => 'AppBundle:Category',
        'choice_label' => 'displayName',
    ));

If left blank, the entity object will be cast to a string and so must have a ``__toString()``
method. You can also pass a callback function for more control::

    use Symfony\Bridge\Doctrine\Form\Type\EntityType;
    // ...

    $builder->add('category', EntityType::class, array(
        'class' => 'AppBundle:Category',
        'choice_label' => function ($category) {
            return $category->getDisplayName();
        }
    ));

The method is called for each entity in the list and passed to the function. For
more details, see the main :ref:`choice_label <reference-form-choice-label>` documentation.

.. note::

    When passing a string, the ``choice_label`` option is a property path. So you
    can use anything supported by the
    :doc:`PropertyAccessor component </components/property_access/introduction>`

    For example, if the translations property is actually an associative
    array of objects, each with a name property, then you could do this::

        use Symfony\Bridge\Doctrine\Form\Type\EntityType;
        // ...

        $builder->add('gender', EntityType::class, array(
           'class' => 'AppBundle:Category',
           'choice_label' => 'translations[en].name',
        ));

class
~~~~~

**type**: ``string`` **required**

The class of your entity (e.g. ``AppBundle:Category``). This can be
a fully-qualified class name (e.g. ``AppBundle\Entity\Category``)
or the short alias name (as shown prior).

em
~~

**type**: ``string`` | ``Doctrine\Common\Persistence\ObjectManager`` **default**: the default entity manager

If specified, this entity manager will be used to load the choices
instead of the ``default`` entity manager.

query_builder
~~~~~~~~~~~~~

**type**: ``Doctrine\ORM\QueryBuilder`` or a Closure **default**: ``null``

Allows you to create a custom query for your choices. See
:ref:`ref-form-entity-query-builder` for an example.

The value of this option can either be a ``QueryBuilder`` object, a Closure or
``null`` (which will load all entities). When using a Closure, you will be
passed the ``EntityRepository`` of the entity as the only argument and should
return a ``QueryBuilder``.

If you'd like to display an empty list of entries, you can return ``null`` in
the Closure.

Overridden Options
------------------

choice_name
~~~~~~~~~~~

.. versionadded:: 2.7
    The ``choice_name`` option was introduced in Symfony 2.7.

**type**: ``string``, ``callable`` or :class:`Symfony\\Component\\PropertyAccess\\PropertyPath` **default**: id

By default the name of each field is the id of the entity, if it can be read
from the class metadata by an internal id reader. Otherwise the process will
fall back to using increasing integers.

choice_value
~~~~~~~~~~~~

.. versionadded:: 2.7
    The ``choice_value`` option was introduced in Symfony 2.7.

**type**: ``string``, ``callable`` or :class:`Symfony\\Component\\PropertyAccess\\PropertyPath` **default**: id

As for the ``choice_name`` option, ``choice_value`` uses the id by default.
It allows an optimization in the :class:``Symfony\\Bridge\\Doctrine\\Form\\ChoiceList\\Loader\\DoctrineChoiceLoader`` which will
only load the ids passed as values while the form submission.
It prevents all non submitted entities to be loaded from the database, even
when defining the ``query_builder`` option.
If it may be useful to set this option using an entity's property as string
value (e.g for some API), you will gain performances by letting this option set
by default.

.. note::

    If the id cannot be read, for BC, the component checks if the class implements
    ``__toString()`` and will use an incremental integer otherwise.

choices
~~~~~~~

**type**:  ``array`` | ``\Traversable`` **default**: ``null``

Instead of allowing the `class`_ and `query_builder`_ options to fetch the
entities to include for you, you can pass the ``choices`` option directly.
See :ref:`reference-forms-entity-choices`.

data_class
~~~~~~~~~~

**type**: ``string`` **default**: ``null``

This option is not used in favor of the ``class`` option which is required
to query the entities.

Inherited Options
-----------------

These options inherit from the :doc:`ChoiceType </reference/forms/types/choice>`:

.. include:: /reference/forms/types/options/choice_attr.rst.inc

.. include:: /reference/forms/types/options/choice_translation_domain.rst.inc

.. include:: /reference/forms/types/options/expanded.rst.inc

.. include:: /reference/forms/types/options/group_by.rst.inc

.. include:: /reference/forms/types/options/multiple.rst.inc

.. note::

    If you are working with a collection of Doctrine entities, it will be
    helpful to read the documentation for the
    :doc:`/reference/forms/types/collection` as well. In addition, there
    is a complete example in the cookbook article
    :doc:`/cookbook/form/form_collections`.

.. include:: /reference/forms/types/options/placeholder.rst.inc

.. include:: /reference/forms/types/options/preferred_choices.rst.inc

.. note::

    This option expects an array of entity objects (that's actually the same as with
    the ``ChoiceType`` field, whichs requires an array of the preferred "values").

.. include:: /reference/forms/types/options/choice_type_translation_domain.rst.inc

These options inherit from the :doc:`form </reference/forms/types/form>`
type:

.. include:: /reference/forms/types/options/data.rst.inc

.. include:: /reference/forms/types/options/disabled.rst.inc

.. include:: /reference/forms/types/options/empty_data.rst.inc
    :end-before: DEFAULT_PLACEHOLDER

The actual default value of this option depends on other field options:

* If ``multiple`` is ``false`` and ``expanded`` is ``false``, then ``''``
  (empty string);
* Otherwise ``array()`` (empty array).

.. include:: /reference/forms/types/options/empty_data.rst.inc
    :start-after: DEFAULT_PLACEHOLDER

.. include:: /reference/forms/types/options/error_bubbling.rst.inc

.. include:: /reference/forms/types/options/error_mapping.rst.inc

.. include:: /reference/forms/types/options/label.rst.inc

.. include:: /reference/forms/types/options/label_attr.rst.inc

.. include:: /reference/forms/types/options/label_format.rst.inc

.. include:: /reference/forms/types/options/mapped.rst.inc

.. include:: /reference/forms/types/options/required.rst.inc
