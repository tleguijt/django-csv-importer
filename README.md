CSV Importer
============

CSV Importer is a Django application which allows developers to import CSV files and map their data to models, mapping column headers to model fields.

Fork info
---------
This project has been forked from girasquid/django-csv-importer in order to make some changes to make this project compatible with Django 1.8


Installation notes
==================

CSV Importer has no external dependencies, other than Django itself and the
contrib.contenttypes application.


It's also recommended that you have ``django.contrib.admin`` installed
for ease of site maintenance - the app even comes with default templates wired
up for adding to the admin.

To install, do a Git checkout of CSV Importer from somewhere
on your Python path:

git clone git://github.com/girasquid/django-csv-importer.git
python django-csv-importer/setup.py install

Then add `csvimporter` to the ``INSTALLED_APPS`` setting of your Django
project, run ``manage.py syncdb``, and either put a call to
``include('csvimporter.urls')`` somewhere in your root URLConf(or copy
over the URL patterns from CSV Importer that you want to use).



Templates
=========

The Git checkout will get you a set of example templates
that hook into the contrib admin site; you can also create templates
with the same names and fill them in however you like.



Available Settings
==================

CSV Importer relies on 4 settings:

	* ``CSVIMPORTER_KEY_TO_FIELD_MAP`` - maps keys from the CSV file's fieldnames
		to fields on the model(Default: lambda k: k.replace(' ','_').lower())
		
	* ``CSVIMPORTER_EXCLUDE`` - lists apps(or app.modelnames) to exclude from the 
		list of available content types to import the CSV as.
		
	* ``CSVIMPORTER_UPLOAD_TO`` - where CSV files that are being imported should be
		stored(Default: csvimporter).
		
	* ``CSVIMPORTER_DATA_TRANSFORMS` - maps transformation functions to models(See
		below, `Transforming Data`)
	

Transforming Data
=================

After importing CSV data(but before saving the model), CSV Importer will call functions
inside the CSVIMPORTER_DATA_TRANSFORMS dict that are keyed to the model that is being
imported. It will pass in the request object, and all of the data that it has gathered
for the model.

Example:

def transform_contact(request, data):
	from addressbook.models import Employer
	# data = {'phone': '000-000-0000', 'first_name': 'foo', 'last_name': 'bar',
	#	'employer': 'Acme Systems Inc.'}
	data['phone']      = parse_phone_number(data['phone'])	# returns '(000) 000-0000'
	data['first_name'] = data['first_name'].title()
	data['last_name']  = data['last_name'].title()
	# this is how you handle foreign keys
	data['employer'] = Employer.objects.get_or_create(name=data['employer'])[0]
	return data

CSVIMPORTER_DATA_TRANSFORMS = {
	'addressbook.contact': transform_contact,
}

In this case, when someone imports a CSV for the addressbook.contact model, the function
transform_contact will be called. It will reformat the phone number, titlecase the first
and last names, and set up the 'employer' foreignkey on the model.