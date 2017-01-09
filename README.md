# Drupal 8 Migration Example

Usually when a huge site makes the (wise) decision to migrate to Drupal, one of the biggest concerns of the site owners is _How to migrate the old site's data into the new Drupal site_. The old site might or might not be a Drupal site, but given that the new site is on Drupal, we can make use of the cool _migrate_ module to import data from a variety of data sources including but not limited to XML, JSON, CSV and SQL databases.

This project is an example module showing how to go about importing basic data from a CSV data source though things would work pretty similarly for other types of data sources. Apart from a basic data import, I have also included certain other important things which a migration might involve, for example, import of 2 different types of entities and the relation between them.

Though being written to serve as a simple example for demonstrating the basics of Drupal 8 migrations, in this project we cover:

* Import of basic content as Drupal _node_ entities or _taxonomy term_ entities.
* Certain ways of basic data manipulation during data the import process.
* Import of basic relationships between two entities, eg, articles and tags.
* Import of images / files as Drupal _file_ entities and relating them to the relevant content.

# Quick start

* Download the files from this repo and put them in the `modules/custom/c11n_migrate` directory. `git clone https://github.com/jigarius/drupal-migration-example.git modules/custom/c11n_migrate`
* Install the module. `drush en c11n_migrate -y`
* See current status of the migrations. `drush migrate-status`
* Run / re-run the migrations introduced by this module. `drush migrate-import --group=c11n --update`

# The problem

As per project requirements, we wish to import certain data for an educational and cultural insitution.

* **Academic programs:** We have a CSV file containing details related to academic programs. We are required to create _nodes_ of type _program_ with the data.
* **Tags:** We have a CSV file containing details related to tags for these academic programs. We are required to import these as _terms_ of the _vocabulary_ named _tags_.
* **Images:** We have images for each academic program. The base name of the images are mentioned in the CSV file for academic programs. To make things easy, we have only one image per program.

# Executing migrations

Before we start with actual migrations, there are certain things which I would point out so as to ensure that you can run your migrations without trouble.

* Though the basic migration framework is a part of the D8 core as the migrate module, to be able to execute migrations, you must install the [migrate_tools](https://www.drupal.org/project/migrate_tools) module. You can use the command drush migrate-import --all to execute all migrations. In this tutorial, we also install some other modules like [migrate_plus](https://www.drupal.org/project/migrate_plus), [migrate_source_csv](https://www.drupal.org/project/migrate_source_csv).
* Migration definitions in Drupal 8 are in YAML files, which is great. But the fact that they are located in the `config/install` directory implies that these YAML files are imported when the module is installed. Hence, any subsequent changes to the YAML files would not be detected untill the module is re-installed. We can to do that by re-importing the relevant configurations like `drush config-import --partial --source=path/to/module/config/install`.
* While writing a migration, you would usually be updating your migration over and over and re-running them to see how things go. So, to do this quickly, you can re-install the module containing your custom migrations (in this case the _c11n_migrate_ module) and execute the relevant migrations in a single command like `drush config-import --partial --source=sites/sandbox.com/modules/c11n_migrate/config/install -y && drush migrate-import --group=c11n --update -y`.
* To execute the migrations in this example, you can download this repo and rename the downloaded directory to c11n_migrate. It should work without issues with a _standard_ Drupal 8 install.

# [c11n_migrate.info.yml](c11n_migrate.info.yml)

I usually prefer to name project-specific custom modules with a prefix of _c11n_ (being the numeronym for customization). This way, have a naming convention for custom modules and I can copy any custom module to another site without worrying about having to change prefixes. The fact to be noted is that we have a _.info.yml_ file instead of the _.info_ file we were used to in Drupal 7.

Nothing fancy about the module file as such. It includes a basic project definition with certain dependencies on other modules. Though the _migrate_ module is in Drupal 8 core, we need most of these dependencies to enable / enhance migrations on the site:
* **migrate**: Without the migrate module, we cannot migrate!
* **migrate_plus**: Improves the core _migrate_ module by adding certain functionality like migration groups and usage of YML files to define migrations. Apart from that this module includes an example module which I referred to on various occasions while writing my example module.
* **migrate_tools**: General-purpose drush commands and basic UI for managing migrations.
* **migrate_source_csv**: The core _migrate_ module provides a basic framework for migrations, which does not include support for specific data sources. This module makes the _migrate_ module work with CSV data sources. There are other modules which provide support for other data sources like JSON, XML, etc.
* **node**: We will be importing _academic programs_ as nodes. Thus, we need the _node_ module.
* **taxonomy**: We will be importing _tags_ as taxonomy terms. Thus, we need the _taxonomy_ module.

# [c11n_migrate.module](c11n_migrate.module)

In Drupal 8, unlike Drupal 7, a module only provides a _.module_ file only if required. In our example, we do not need the `.module` file, so I have not created one.

# Data source

Ref: [import/README.md](import/README.md)

To import the data, we would obviously require a data source. For the sake of this example, I have provided the source data in a _import_ directory inside the module and the data is copied to the `public://` directory using `hook_install`.

# Migration group

Ref: [migrate_plus.migration_group.c11n.yml](config/install/migrate_plus.migration_group.c11n.yml)

Like we used to implement _hook_migrate_api()_ in Drupal 7 to declare the API version, migration groups, individual migrations and more, in Drupal 8, we do something similar. Instead of implementing a hook, we create a migration group declaration inside the _config/install_ directory of our module. The file must be named something like _migrate_plus.migration_group.NAME.yml_ where _NAME_ is the machine name for the migration group.

In this example, we define a migration group _c11n_ to provide general information like:

* **id:** A unique ID for the migration. This is usually the _NAME_ part of the migration group declaration file name as discussed above.
* **label:** A human-friendly name of the migration group as it would in the UI.
* **description:** A brief description about the migration group.
* **source_type:** This would appear in the UI to provide a general hint as to where the data for this migration comes from.
* **dependencies:** Though this might sound a bit strange for Drupal 7 users (like me), this segment is used to define modules on which the migration depends. When one of these required modules are missing / removed, the migration group is automatically removed.

We can execute all migrations in a given group with the command `drush migrate-import --group=GROUP`.

# Migration definition: Metadata

Ref: [migrate_plus.migration.program_data.yml](config/install/migrate_plus.migration.program_data.yml)

Now that we have a module to put our migration scripts in and a migration group for grouping them together, it's time we write a basic migration! To get started, we import basic data about academic programs, ignoring complex stuff such as tags, files, etc. In Drupal 7 we used to do this in a file containing a PHP class which used to extend the _Migration_ class provided by the _migrate_ module. In Drupal 8, like many other things, we do this in a YML file.

In migration declaration file, we declare some metadata about the migration:

* **id:** A unique identifier for the migration. In this example, I allocated the ID _program_data_, hence, the migration declaration file has been named migrate_plus.migration._program_data_.yml. We can execute specific migrations with the command `drush migrate-import ID`.
* **label:** A human-friendly name of the migration as it would appear in the UI.
* **migration_group:** This puts the migration into the migration group _c11n_ we created above. We can execute all migrations in a given group with the command `drush migrate-import --group=GROUP`.
* **migration_tags:** Here we provide multiple tags for the migration and just like groups, we can execute all migrations with the same tag using the command `drush migrate-import --tag=TAG`
* **dependencies:** Just like in case of migration groups, this segment is used to define modules on which the migration depends. When one of these required modules are missing / removed, the migration is automatically removed.
* **migration_dependencies:** This element is used to mention IDs of other migrations which must be run before this migration. For example, if we are importing articles and their authors, we need to import author data first so that we can refer to the author's ID while importing the articles. Note that we can leave this undefined for now as we do not have any other migrations defined. I defined this section only after I finished writing the migrations for tags, files, etc.

# Migration definition: Source

Ref: [migrate_plus.migration.program_data.yml](config/install/migrate_plus.migration.program_data.yml)

Once done with the meta-data, we define the source of the migration data with the _source_ element in the YAML.

* **plugin:** The plugin responsible for reading the source data. In our case we use the _migrate_source_csv_ module which provides the source plugin _csv_.
* **path:** Path to the data source file - in this case, the [program.data.csv](import/program/program.data.csv) file.
* **header_row_count:** This is a plugin-specific parameter which allows us to skip a number of rows from the top of the CSV. I found this parameter in the plugin class file, but I'm sure it must also be mentioned in the documentation for the _migrate_source_csv_ module.
* **keys:** This parameter defines a number of columns in the source data which form a unique key in the source data. Luckily in our case, the program.data.csv provides a unique ID column so things get easy for us in this migration. This unique key will be used by the migrate module to relate records from the source with the records created in our Drupal site. With this relation, the migrate module can interpret changes in the source data and update the relevant data on the site. To execute an update, we use the parameter `--update` with our `drush migrate-import` command.
* **fields:** This parameter defines provides a description for the various columns available in the CSV data source. These descriptions just appear in the UI and explain purpose behind each column of the CSV.

# Migration definition: Destination

Ref: [migrate_plus.migration.program_data.yml](config/install/migrate_plus.migration.program_data.yml)

Similarly, we need to tell the migrate module how we want it to use the source data. We do this with the _destination_ element in the YAML.

* **plugin:** Just like source data is handled by separate plugins, we have _destination_ plugins to handle the output of the migrations. In this case, we want Drupal to create _node_ entities with the academic program data, so we use the _node_ plugin.
* **default_bundle:** Here, we define the type of nodes we wish to obtain using the migration. Though we can override the bundle for individual imports, this parameter provides a default bundle for entities created by this migration. We will be creating only _program_ nodes, so we mention that here.

# Migration definition: Mapping and processing

Ref: [migrate_plus.migration.program_data.yml](config/install/migrate_plus.migration.program_data.yml)

If you ever wrote a migration in an earlier version of Drupal, you might already know that migration are usually not as simple as copying data from one column of a CSV file to a given property of the relevant entity. We need to process certain columns and eliminate certain columns and much more. In Drupal 8, we define these processes using a _process_ element in the migration declaration. This is where we put our YAML skills to real use.

* **title:** An easy property to start with, we just assign the _Title_ column of the CSV as the _title_ property of the node.
* **sticky:** Though Drupal can apply the default value for this property if we skip it, I wanted to demonstrate how to specify a default value for a property. We use the _default_value_ plugin with the _default_value_ parameter to make the imports non-sticky with _sticky = 0_.
* **uid:** Similarly we specify default owner for the article as the administrative user with _uid = 1_.
* **body:** The _body_ is a filtered long text field and has various sub-properties we can set. So, we copy the _Body_ column from the CSV file to the _body/value_ property (instead of assigning it to just _body_). In the next line, we specify the _body/format_ property as _restricted_html_. Similary, one can also add a custom summary for the nodes using the _body/summary_ property. However, we should keep in mind that while defining these sub-properties, we need to wrap the property name in quotes because we have a `/` in the property name.
* **field_program_level:** With this property we get to try out the useful _static_map_ plugin. Here, the source data uses the values _graduate/undergraduate_ whereas the destination field only accepts _gr/ug_. In Drupal 7, we would have written a few lines of code in a _ProgramDataMigration::prepareRow()_ method, but in Drupal 8, we just write some more YAML. Here, we have the plugin specifications as usual, but we have small dashes with which we are actually defining an array of plugins or a _plugin pipeline_. With the first plugin, we call the function _strtolower_ (with `callback: strtolower`) on the _Level_ property (with `source: Level`). Once the old value is in lower case, we pass it through the _static_map_ (with `plugin: static_map`) and define a map of new values which should be used instead of old values (with the `map` element). Done!

With the parameters above, we can write basic migrations with basic data-manipulation. If you wish to see another basic migration, you can take a look at [migrate_plus.migration.program_tags.yml](config/install/migrate_plus.migration.program_tags.yml). In the sections below, I would explain how to do some complex tasks like importing taxonomy terms and their relations with nodes and uploading files / images and associating them with their relevant nodes.

# Migrating taxonomy terms

This section is about migrating relations between two entities. Before jumping to this section, one must ensure that a migration has already been written to import the target entities. In our example, we wish to associate _tags_ (taxonomy terms) to _academic programs_ (nodes). For that, we need to import taxonomy terms and we do that in [migrate_plus.migration.program_tags.yml](config/install/migrate_plus.migration.program_tags.yml). In the migration for _tags_ data, we use the tag text as a unique key for the tags. This is because:

* The tags data-source, [program.tags.csv](import/program/program.tags.csv), does not provide any unique key for the tags.
* The academic programs data-source, [program.data.csv](import/program/program.data.csv), refers to the tags using the tag text (instead of unique IDs).

Once the tag data is imported, all we have to do is add some simple lines of YAML in the [migration definition for academic programs](config/install/migrate_plus.migration.program_data.yml) to tell Drupal how to migrate the _field_tags_ property of academic programs. As we did above for _program_level_ we will be specifying multiple plugins for this property:

* **explode:** Taking a look at the data-source, we notice that academic programs have multiple tags separated by commas. So, as a first step, we use the explode plugin which would split/explode the tags by the delimiter _, (comma)_, thereby creating an array of tags. We do this using `plugin: explode` and `delimiter: ', '`.
* **migration:** Now that we have an array of tags, each tag identifying itself using it's unique tag text, we tell the migrate module that these tags are the same ones we imported in _migrate_plus.migration.program_tags.yml_ and that the tags generated during that migration are to be used here in order to associate them to the academic programs. We do this using `plugin: migration` and `migration: program_tags`.

As simple as it may sound, this is all that is needed to associate the tags to the academic programs!

For the sake of demonstration, I also included an alternative approach for the migration of the _field_program_type_ property. For program type, I used the entity_generate plugin, which does the following:

* Looks up for an entity of a particular type (in this case, _taxonomy_term_) based on a particular property (in this case, _name_).
* If no matching entity is found, an entity is created on the fly.
* The ID of the existing / created entity is returned for use in the migration.

So, in the _process_ instructions for _field_program_type_, I use `plugin: entity_generate`. So, during migration, for every _program type_, the entity_generate plugin is called and a particular taxonomy term is associated to the academic programs. The disadvantage of using the entity_generate method is when we rollback the migration, these taxonomy terms created during the migration would not be deleted.

To make sure that tag data is imported and available during the academic program migration, we specify the `program_tags` migration in the `migration_dependencies` for the `program_data` migration. Now, when you re-run these migrations, the taxonomy terms get associated to the academic program nodes.

# Migrating files / images

In our example, every academic program has an associated image file. Say, the client wants us to associate these files to the academic programs created during the migration. Though it might sound difficult, the solution involves only two steps:

**Step 1, migrate files**

Ref: [config/install/migrate_plus.migration.program_image.yml](config/install/migrate_plus.migration.program_image.yml)

Like we did with the taxonomy terms above, first we need to create _file_ entities for each file. This is because Drupal treats files as _file_ entities which have their own ID and then Drupal treats node-file associations as entity references, referring to the file entities with their IDs.

We create the file entities in the [migrate_plus.migration.program_image.yml](config/install/migrate_plus.migration.program_image.yml) file, but this time, using some other process plugins. Following are some important notes on the program_image migration:

* We specify the _key_ parameter in _source_ as the column containing file names, ie, _Image file_. This way, we would be refer to these files in other migrations using their names, eg, `engineering.png`.
* We mention an additional parameter _constants_ in the _source_ element.
  * _file_source_uri_ is used to refer to the path from which files are to be read during the import.
  * _file_dest_uri_ is used to refer to the destination path where files should be copied to. The newly created file entities would refer to files stored in this directory.
* The `public://` URI refers to the _files_ directory inside the site in question. This is where all public files related to the site are stored.
* In the _process_ element, we prepare two paths - the file source path and the file destination path.
  * _file_source_ is obtained by concatenating the _file_source_uri_ with the _Image file_ column which stores the file's basename. Using `delimiter: /` we tell the migrate module to join the two strings with a `/ (slash)` in between to ensure we have a valid file name. In short, we do `file_source_uri . '/' . basename` using the `concat` plugin.
  * _file_dest_, in a similar way, is `file_dest_uri . '/' . basename`. This is where we utilize the constants we defined in the _source_ element.
* Now, we use the _file_source_ and _file_dest_ paths generated above with `plugin: file_copy`. The _file_copy_ plugin simply copies the files from the `file_source` path to the `file_dest` path. All the steps we did above were just for being able to copy the files.
* Finally, since the _destination_ of the migration is `entity:file`, the migrate module would use the file created in the previous step to generate a _file_ entity, thereby generating a unique file ID.

**Step 2, associate files:**

Once the heavy-lifting is done and we have our file entities, we need to put the files to use by associating them to academic programs. To do this, we write add processing instructions for `file_image` in [migration_plus.migration.program_data.yml](config/install/migration_plus.migration.program_data.yml). Just like we did for taxonomy terms, we tell the migrate module that the _Image file_ column contains a unique file name, which refers to a file entity created during the _program_image_ migration. Hence, we write `plugin: migration` and `migration: program_image`. And it's done!

To make sure that tag data is imported and available during the academic program migration, we specify the `program_image` migration in the `migration_dependencies` for the `program_data` migration. Now, when you run these migrations, the image files get associated to the academic program nodes.
