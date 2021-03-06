Data Fixtures
=============

The Sulu DocumentManager integration includes a fixture loader which allows
you to load static data into your content repository.

Getting Started
---------------

Shown below is an example that creates a simple data fixture.

**Note**
    - The class name MUST end with `Fixture` for it to be recognized
    - The class MUST be placed in `<your bundle>/DataFixtures/Document` in order
      for it to be loaded automatically.

.. code-block:: php

    <?php
    namespace YourBundle\DataFixtures\Document;

    use Sulu\Component\DocumentManager\DocumentManager;
    use Sulu\Bundle\DocumentManagerBundle\DataFixtures\DocumentFixtureInterface;
    use Sulu\Component\DocumentManager\Exception\MetadataNotFoundException;

    class SomeFixture implements DocumentFixtureInterface
    {
        /**
         * Simple local string with two chars.
         */
        const LOCALE = 'en';

        /**
         * All fixtures will be sorted in regards of the returned integer. This
         * "weight" will let a fixture run later if the integer is higher.
         *
         * @return int
         */
        public function getOrder()
        {
            return 10;
        }

        /**
         * Load fixtures.
         *
         * Use the document manager to create and save fixtures.
         * Be sure to call DocumentManager#flush() when you are done.
         *
         * @param DocumentManager $documentManager
         *
         * @throws MetadataNotFoundException
         */
        public function load(DocumentManager $documentManager)
        {
            /**
             * "page" is the base content of sulu. "article" for example would be used be the Article bundle.
             *
             * @var \Sulu\Bundle\PageBundle\Document\PageDocument $document
             */
            $document = $documentManager->create('page');

            // Set the local. Keep in mind that you have to save every local version extra.
            $document->setLocale(static::LOCALE);

            // The title of the page set in the template XML. Can not be set by getStructure()->bind();
            $document->setTitle('foo bar page title');

            // Use setStructureType to set the name of the page template.
            $document->setStructureType('default');

            // URL of the content with out any language prefix.
            $document->setResourceSegment('/foo-bar-page');

            // Data for all content types that this template uses.
            $document->getStructure()->bind(
                [
                    'article' => '<strong>Lore Ipsum Dolor</strong>',
                ]
            );

            // Data for the "Excerpt & Taxonomies" tab when editing content.
            $document->setExtension(
                'excerpt',
                [
                    'title' => 'foo title',
                    'description' => 'bar description',
                    'categories' => [],
                    'tags' => [],
                ]
            );

            // Data for the "SEO" tab when editing content.
            $document->setExtension(
                'seo',
                [
                    'title' => 'foo title',
                ]
            );

            // parent_path uses your webspace name. In this case "sulu_io"
            $documentManager->persist(
                $document,
                static::LOCALE,
                [
                    'parent_path' => '/cmf/sulu_io/contents',
                ]
            );

            // Optional: If you don't want your document to be published, remove this line
            $documentManager->publish($document, static::LOCALE);

            // Persist immediately to database.
            $documentManager->flush();
        }
    }

You can now execute your data fixture using the
``sulu:document:fixtures:load``
command.

.. code-block:: bash

    $ php bin/console sulu:document:fixtures:load

By default this command will purge and re-initialize the workspace before
loading all of the fixtures.

.. warning::

    Unless you use the `--append` option, your workspace will be purged!

Advanced Usage
--------------

You can specify directories instead of having the command automatically find
the fixtures:

.. code-block:: bash

    $ php bin/console sulu:document:fixtures:load --fixtures=/path/to/fixtures1 --fixtures=/path/to/fixtures2

You can also specify if fixtures should be *appended* (i.e. the repository will
not be purged) and if the initializer should be executed.

Append fixtures:

.. code-block:: bash

    $ php bin/console sulu:document:fixtures:load --append

Do not initialize:

.. code-block:: bash

    $ php bin/console sulu:document:fixtures:load --no-initialize

Using the Service Container
---------------------------

If you need the service container you can implement the `Symfony\Component\DependencyInjection\ContainerAwareInterface`:

.. code-block:: php

    <?php

    namespace YourBundle\DataFixtures\Document;

    use Sulu\Bundle\DocumentManagerBundle\DataFixtures\DocumentFixtureInterface;
    use Symfony\Component\DependencyInjection\ContainerAwareInterface;
    use Symfony\Component\DependencyInjection\ContainerInterface;

    class SomeFixture implements DocumentFixtureInterface, ContainerAwareInterface
    {
        private $container;

        public function setContainer(ContainerInterface $container = null)
        {
            $this->container = $container;
        }
    }
