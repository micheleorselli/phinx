.. index::
   single: Writing Migrations

Writing Migrations
==================

Phinx relies on migrations in order to transform your database. Each migration
is represented by a PHP class in a unique file. It is preferred that you write
your migrations using the Phinx PHP API, but a raw SQL is also supported.

Creating a New Migration
------------------------

Let's start by creating a new Phinx migration. Run Phinx using the
``create`` command.

.. code-block:: bash
    
        $ phinx create MyNewMigration
        
This will create a new migration in the format
``YYYYMMDDHHMMSS_my_new_migration.php`` where the first 14 characters are
replaced with the current timestamp down to the second.

Phinx automatically creates a skeleton migration file with two empty methods.

.. code-block:: php
        
        <?php

        use Phinx\Migration\AbstractMigration;

        class MyNewMigration extends AbstractMigration
        {
            /**
             * Migrate Up.
             */
            public function up()
            {
            
            }

            /**
             * Migrate Down.
             */
            public function down()
            {

            }
        }

The AbstractMigration Class
---------------------------

All Phinx migrations extend from the ``AbstractMigration`` class. This class
provides the necessary support to create your database migrations. Database
migrations can transform your database in many ways such as creating new
tables, inserting rows, adding indexes and modifying columns.

The Up Method
~~~~~~~~~~~~~

The up method is automatically run by Phinx when you are migrating up and it
detects the given migration hasn't been executed previously. You should use the
up method to transform the database with your intended changes.

The Down Method
~~~~~~~~~~~~~~~

The down method is automatically run by Phinx when you are migrating down and
it detects the given migration has been executed in the past. You should use
the down method to reverse/undo the transformations described in the up method.

The Change Method
~~~~~~~~~~~~~~~~~

Phinx 0.2.0 introduced a new feature called reversible migrations. With
reversible migrations you only need to define the ``up`` logic and Phinx can
figure out how to migrate down automatically for you. To define a reversible
migration you must declare a ``change`` method in your migration file. For
example:

.. code-block:: php
        
        <?php

        use Phinx\Migration\AbstractMigration;

        class CreateUserLoginsTable extends AbstractMigration
        {
            /**
             * Change.
             */
            public function change()
            {
                // create the table
                $table = $this->table('user_logins');
                $table->addColumn('user_id', 'integer')
                      ->addColumn('created', 'datetime')
                      ->create();
            }
    
            /**
             * Migrate Up.
             */
            public function up()
            {
    
            }

            /**
             * Migrate Down.
             */
            public function down()
            {

            }
        }

When executing this migration Phinx will create the ``user_logins`` table on
the way up and automatically figure out how to drop the table on the way down.

.. note::

    When creating or updating tables inside a ``change()`` method you must use
    the Table ``create()`` and ``update()`` methods. Phinx cannot automatically
    determine whether a ``save()`` call is creating a new table or modifying an
    existing one.

Phinx can only reverse the following commands:

-  createTable
-  renameTable
-  addColumn
-  renameColumn
-  addIndex
-  addForeignKey

If a command cannot be reversed then Phinx will throw a 
``IrreversibleMigrationException`` exception when it's migrating down.

Executing Queries
-----------------

Queries can be executed with the ``execute()`` and ``query()`` methods. The
``execute()`` method returns the number of affected rows whereas the
``query()`` method returns the result as an array.

.. code-block:: php
        
        <?php
        
        // execute()
        $count = $this->execute('DELETE FROM users'); // returns the number of affected rows

        // query()
        $rows = $this->query('SELECT * FROM users'); // returns the result as an array

Fetching Rows
-------------

There are two methods available to fetch rows. The ``fetchRow()`` method will
fetch a single row, whilst the ``fetchAll()`` method will return multiple rows.
Both methods accept raw SQL as their only parameter.

.. code-block:: php
        
        <?php
        
        // fetch a user
        $row = $this->fetchRow('SELECT * FROM users');

        // fetch an array of messages
        $rows = $this->fetchAll('SELECT * FROM messages');

Working With Tables
-------------------

The Table Object
~~~~~~~~~~~~~~~~

The Table object is one of the most useful APIs provided by Phinx. It allows
you to easily manipulate database tables using PHP code. You can retrieve an
instance of the Table object by calling the ``table()`` method from within
your database migration.

.. code-block:: php
    
        <?php
        
        $table = $this->table('tableName');

You can then manipulate this table using the methods provided by the Table
object.

Creating a Table
~~~~~~~~~~~~~~~~

Creating a table is really easy using the Table object. Let's create a table to
store a collection of users.

.. code-block:: php

        <?php
        
        $users = $this->table('users');
        $users->addColumn('username', 'string', array('limit' => 20))
              ->addColumn('password', 'string', array('limit' => 40))
              ->addColumn('password_salt', 'string', array('limit' => 40))
              ->addColumn('email', 'string', array('limit' => 100))
              ->addColumn('first_name', 'string', array('limit' => 30))
              ->addColumn('last_name', 'string', array('limit' => 30))
              ->addColumn('created', 'datetime')
              ->addColumn('updated', 'datetime', array('default' => null))
              ->addIndex(array('username', 'email'), array('unique' => true))
              ->save();
        
Columns are added using the ``addColumn()`` method. We create a unique index
for both the username and email columns using the ``addIndex()`` method.
Finally calling ``save()`` commits the changes to the database.

.. note::

    Phinx automatically creates an auto-incrementing primary key for every
    table called ``id``.

To specify an alternate primary key you can specify the ``primary_key`` option
when accessing the Table object. Let's disable the automatic ``id`` column and
create a primary key using two columns instead:

.. code-block:: php

        <?php
        
        $table = $this->table('followers', array('id' => false, 'primary_key' => array('user_id', 'follower_id')));
        $table->addColumn('user_id', 'integer')
              ->addColumn('follower_id', 'integer')
              ->addColumn('created', 'datetime')
              ->save();

Setting a single ``primary_key`` doesn't enable the ``AUTO_INCREMENT`` option.
To do this, we need to override the default ``id`` field name:

.. code-block:: php

        <?php

        $table = $this->table('followers', array('id' => 'user_id'));
        $table->addColumn('user_id', 'integer')
              ->addColumn('follower_id', 'integer')
              ->addColumn('created', 'datetime')
              ->save();

Determining Whether a Table Exists
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You can determine whether or not a table exists by using the ``hasTable()``
method.

.. code-block:: php

        <?php
        
        $exists = $this->hasTable('users');
        if ($exists) {
            // do something
        }

Dropping a Table
~~~~~~~~~~~~~~~~

Tables can be dropped quite easily using the ``dropTable()`` method.

.. code-block:: php
        
        <?php
        
        $this->dropTable('tableName');
        
Renaming a Table
~~~~~~~~~~~~~~~~

To rename a table access an instance of the Table object then call the
``rename()`` method.

.. code-block:: php
        
        <?php
        
        $table = $this->table('users');
        $table->rename('legacy_users');

Working With Columns
~~~~~~~~~~~~~~~~~~~~

Renaming a Column
~~~~~~~~~~~~~~~~~

To rename a column access an instance of the Table object then call the
``renameColumn()`` method.

.. code-block:: php
        
        <?php

        $table = $this->table('users');
        $table->renameColumn('bio', 'biography');

Working With Foreign Keys
~~~~~~~~~~~~~~~~~~~~~~~~~

Phinx has support for creating foreign key constraints on your database tables.
Let's add a foreign key to an example table:

.. code-block:: php

        <?php
        
        $table = $this->table('tags');
        $table->addColumn('tag_name', 'string')
              ->save();
        
        $refTable = $this->table('tag_relationships');
        $refTable->addColumn('tag_id', 'integer')
                 ->save();
                
        $refTable->addForeignKey('tag_id', 'tags', 'id');

We can also easily check if a foreign key exists:

.. code-block:: php

        <?php
        
        $table = $this->table('tag_relationships');
        $exists = $table->hasForeignKey('tag_id');
        if ($exists) {
            // do something
        }

Finally to delete a foreign key use the ``dropForeignKey`` method.

.. code-block:: php

        <?php
        
        $table = $this->table('tag_relationships');
        $table->dropForeignKey('tag_id');

The Save Method
~~~~~~~~~~~~~~~

When working with the Table object Phinx stores certain operations in a
pending changes cache.

When in doubt it is recommended you call this method. It will commit any
pending changes to the database.