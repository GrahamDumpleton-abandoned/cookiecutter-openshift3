# cookiecutter-openshift

This is an experimental Cookiecutter template for creating OpenShift configurations for basic applications.

## Cookiecutter version

Note that this will not work with the current released version of ``cookiecutter``. This is because to implement support for optional data inputs, it was necessary to modify ``cookiecutter``. The claim is to try and get the changes accepted back into ``cookiecutter``, but for now you will need to use a forked version of ``cookiecutter``.

To install the forked version of ``cookiecutter``, preferably in a Python virtual environment, run:

```
pip install -U https://github.com/GrahamDumpleton/cookiecutter/archive/master.zip
```

## Generating a configuration

Once ``cookiecutter`` has been installed, you should run the command:

```
cookiecutter https://github.com/GrahamDumpleton/cookiecutter-openshift.git
```

This will download a copy of the template repository to your local box and then run ``cookiecutter`` on it.

This will prompt you with various questions about what you require for your application under OpenShift. The initial ``repo_name`` value should be set to the directory into which you want the configuration generated. The final result will be a ``project.json`` file within the directory named by ``repo_name``. There will also be ``components`` and ``datastores`` directories, but these can be ignored.

## Sample output for Wagtail CMS

Sample output for creating an OpenShift configuration for running a Wagtail CMS site, including persistent database is included below. Note that this uses the ``warpdrive`` based Python S2I builder for OpenShift and not the default OpenShift Python S2I builder.

```
$ cookiecutter https://github.com/GrahamDumpleton/cookiecutter-openshift.git
Cloning into 'cookiecutter-openshift'...
remote: Counting objects: 15, done.
remote: Compressing objects: 100% (11/11), done.
remote: Total 15 (delta 4), reused 15 (delta 4), pack-reused 0
Unpacking objects: 100% (15/15), done.
Checking connectivity... done.
repo_name [openshift]:
application_name [my-application]: wagtail
git_repository []: https://github.com/GrahamDumpleton/wagtail-demo-site.git
git_repository_branch [master]:
Select language_runtime:
1 - NodeJS
2 - PHP
3 - Python
4 - Ruby
Choose from 1, 2, 3, 4 [1]: 3
Select s2i_builder_image:
1 - openshift/python:2.7
2 - openshift/python:3.3
3 - openshift/python:3.4
4 - ?
Choose from 1, 2, 3, 4 [1]: 4
s2i_builder_image []: grahamdumpleton/warp0-debian8-python27
Select python_web_framework:
1 - Django
2 - ?
Choose from 1, 2 [1]:
django_settings_module []: demo.settings.production
application_replicas [1]: 2
Select requires_external_route:
1 - y
2 - n
Choose from 1, 2 [1]:
route_hostname []:
application_memory_limit [128Mi]:
Select requires_persistent_storage:
1 - y
2 - n
Choose from 1, 2 [1]: 2
Select requires_database:
1 - y
2 - n
Choose from 1, 2 [1]:
Select database_type:
1 - MySQL
2 - PostgreSQL
Choose from 1, 2 [1]: 2
database_user [user3TIC]:
database_password [V7z3RHSvdIhYQQAc]:
database_name [wagtail_db]:
database_memory_limit [128Mi]:
database_volume_capacity [512Mi]:
Select requires_redis:
1 - y
2 - n
Choose from 1, 2 [1]:
redis_memory_limit [128Mi]:
```

To actually deploy the Wagtail CMS site using the generated configuration, use the ``oc create`` command on the ``project.json`` file which was created.

```
$ oc create -f openshift/project.json
imagestream "wagtail" created
imagestream "wagtail-s2i" created
buildconfig "wagtail" created
deploymentconfig "wagtail" created
service "wagtail" created
route "wagtail" created
persistentvolumeclaim "wagtail-db-pvc" created
deploymentconfig "wagtail-db" created
service "wagtail-db" created
imagestream "wagtail-redis" created
deploymentconfig "wagtail-redis" created
service "wagtail-redis" created
```

For this particular example, once deployed and running, you will need to initialise the database and create a super user account for the Django admin. To do this, identify any of the pods for the running instances of the Wagtail CMS site and run Django database migrations and super user creation admin commands.

```
$ oc get pods --selector app=wagtail
NAME              READY     STATUS    RESTARTS   AGE
wagtail-1-369kz   1/1       Running   0          51s
wagtail-1-gfd2j   1/1       Running   0          55s
grumpy-old-man:cookiecutter-openshift graham$ oc rsh wagtail-1-369kz warpdrive exec python manage.py migrate
Operations to perform:
  Apply all migrations: taggit, wagtailredirects, auth, contenttypes, wagtailembeds, admin, wagtailsearch, home, wagtailadmin, sessions, wagtailimages, wagtailforms, wagtailusers, wagtailcore, wagtaildocs
Running migrations:
  Rendering model states... DONE
  Applying contenttypes.0001_initial... OK
  Applying auth.0001_initial... OK
  Applying admin.0001_initial... OK
  Applying admin.0002_logentry_remove_auto_add... OK
  Applying contenttypes.0002_remove_content_type_name... OK
  Applying auth.0002_alter_permission_name_max_length... OK
  Applying auth.0003_alter_user_email_max_length... OK
  Applying auth.0004_alter_user_username_opts... OK
  Applying auth.0005_alter_user_last_login_null... OK
  Applying auth.0006_require_contenttypes_0002... OK
  Applying auth.0007_alter_validators_add_error_messages... OK
  Applying wagtailcore.0001_squashed_0016_change_page_url_path_to_text_field... OK
  Applying wagtailcore.0017_change_edit_page_permission_description... OK
  Applying wagtailcore.0018_pagerevision_submitted_for_moderation_index... OK
  Applying wagtailcore.0019_verbose_names_cleanup... OK
  Applying wagtailcore.0020_add_index_on_page_first_published_at... OK
  Applying wagtailcore.0021_capitalizeverbose... OK
  Applying wagtailcore.0022_add_site_name... OK
  Applying wagtailcore.0023_alter_page_revision_on_delete_behaviour... OK
  Applying home.0001_initial... OK
  Applying home.0002_create_homepage... OK
  Applying home.0003_homepage_body... OK
  Applying sessions.0001_initial... OK
  Applying taggit.0001_initial... OK
  Applying taggit.0002_auto_20150616_2121... OK
  Applying wagtailadmin.0001_create_admin_access_permissions... OK
  Applying wagtaildocs.0001_initial... OK
  Applying wagtaildocs.0002_initial_data... OK
  Applying wagtaildocs.0003_add_verbose_names... OK
  Applying wagtaildocs.0004_capitalizeverbose... OK
  Applying wagtailembeds.0001_initial... OK
  Applying wagtailembeds.0002_add_verbose_names... OK
  Applying wagtailembeds.0003_capitalizeverbose... OK
  Applying wagtailforms.0001_initial... OK
  Applying wagtailforms.0002_add_verbose_names... OK
  Applying wagtailforms.0003_capitalizeverbose... OK
  Applying wagtailimages.0001_initial... OK
  Applying wagtailimages.0002_initial_data... OK
  Applying wagtailimages.0003_fix_focal_point_fields... OK
  Applying wagtailimages.0004_make_focal_point_key_not_nullable... OK
  Applying wagtailimages.0005_make_filter_spec_unique... OK
  Applying wagtailimages.0006_add_verbose_names... OK
  Applying wagtailimages.0007_image_file_size... OK
  Applying wagtailimages.0008_image_created_at_index... OK
  Applying wagtailimages.0009_capitalizeverbose... OK
  Applying wagtailimages.0010_change_on_delete_behaviour... OK
  Applying wagtailredirects.0001_initial... OK
  Applying wagtailredirects.0002_add_verbose_names... OK
  Applying wagtailredirects.0003_make_site_field_editable... OK
  Applying wagtailredirects.0004_set_unique_on_path_and_site... OK
  Applying wagtailredirects.0005_capitalizeverbose... OK
  Applying wagtailsearch.0001_initial... OK
  Applying wagtailsearch.0002_add_verbose_names... OK
  Applying wagtailsearch.0003_remove_editors_pick... OK
  Applying wagtailusers.0001_initial... OK
  Applying wagtailusers.0002_add_verbose_name_on_userprofile... OK
  Applying wagtailusers.0003_add_verbose_names... OK
  Applying wagtailusers.0004_capitalizeverbose... OK
  
$ oc rsh wagtail-1-369kz warpdrive exec python manage.py createsuperuser
Username (leave blank to use 'warpdrive'): grumpy
Email address: grumpy@example.com
Password:
Password (again):
Superuser created successfully.
```

You can then access the ``/admin`` URL of the Wagtail CMS site, login using the super user account you created, and start creating pages.