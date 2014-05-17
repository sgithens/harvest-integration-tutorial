# harvest-integration-tutorial

A tutorial on how to add Harvest functionality to an existing web application.

This is meant to act as a complete guide on how to integrate Harvest into an
existing Django application, with all the nuts and bolts steps. Harvest is
an incredibly useful library for exploring your data, however, it does have
a lot of moving parts so the initial experience of putting it into an 
existing application (ie. not using the harvest init wizard for a new app)
can be a bit daunting at first. These instructions will assume a basic
familiarity with Django development and python development ( you know how
to use virtualenv and pip ). It does not assume you've used backbone,
marionette, or any of the python frameworks harvest depends on.

## A quick overview

If you've read some of the harvest documents or tutorials you may have a 
sense of how all the components work. But before we dive in too deeply
here is a basic overview of the steps you need to go through to add
Harvest in to your django application.

* Install the python deps. This includes Modeltree, Avocado, Serrano, 
  and some small but really useful python libraries Byron has written, 
  that you'll likely want to use else where once you see them.
* Install the HTML5 deps. This just consists of cilantro.
* Add the harvest Django apps to settings.py
* Add the modeltree config to settings.py
* Include Serrano api endpoints in a urls.py
* Add some entries to a urls.py for query, results, and other cilantro
  endpoints.
* Create a template and view that will essentially be the same for 
  all those cilantro endpoints.
* Run manage.py migrate to install the avocado and serrano tables.
* Run manage.py avocado init for at least one of your apps to populate
  the metadata.
* Make a default dataview, datacontext, and datacategory. (I'm not sure
  if this is necessary, but it might have fixed an issue that had
  shown up.)
* At this point you should be able to explore your data!

## Install the python dependencies

Harvest is a fast moving project, and it's built for customization
so I'll show how to install the prepackaged code, and how to fetch
everything if you want to do development, tinker, and debug. Also,
because it's moving fast, I'll leave out version numbers for the
rest of this document, and just assume they are somewhat recent
from 2nd quarter 2014.

### Fast Way

You should be able to just run

    pip install serrano

which should be at the top of the dependency chain and will fetch
all the rest of the dependencies.

### For development, debugging, and tinkering

TODO

## Install the HTML5 deps

### The fast way

From the cilantro release github page, download the latest release
untargzip it, and put it in your django static area.

### For development

TODO

## Add to Installed Apps

In the appropriate settings.py for your setup, add these 4 entries
to your INSTALLED_APP. ( You may already have south from other 
apps )

    INSTALLED_APPS = (
        'south',
        'serrano',
        'avocado',
        'modeltree',
    )

## Add the modeltree

Your root model tree will be the Django model/table with the primary
key you want mounted and relationships traversed for all the searches.
Harvest does the work of exploring foreign key relationships with
this root model. 

It's important to note that you need to chop off the 'models' part
of the path, and maybe also the prefix of your app if it includes a dot.
For instance, below, the django app for this is actually 'ktb.anno',
and the full path to the Donation model class is 'ktb.anno.models.Donation'.

I believe this is a Django convention and may change in the future. Just
a heads up.

    MODELTREES = {
        'default': {
            'model': 'anno.Donation',
        }
    }

TODO Multiple model trees.

## Include the Serrano endpoints

All communication with Harvest on the server happens through RESTful
web services in the Serrano module, and so you need to include it in
at least one of the urls.py of your project.

    url(r'^serrano/', include('serrano.urls')),

## Add urls for query, results, workspace

In one of your apps you'll want to create these urls. In theory,
if you know how to use backbone routes and marionnete you can 
write your own main.js, but starting out it will save you work
to just make these. At the end of the day they will all point to
the same view, and while using harvest the page probably won't 
actually reload because the js just updates the page content
and url in place, but the application needs to be able to land
at any one of them to start out.

    url(r'^query/$', views.harvest, name="harvest"), 
    url(r'^results/$', views.harvest, name="harvest"), 
    url(r'^workspace/$', views.harvest, name="harvest"), 

Just to clarify, you can call your view whatever you'd like,
it's just important that the urls be query, results, and workspace.

## Create a template for them

Next you'll create a template for the above urls and a bare-ish
view for them. At the time of writing, cilantro uses bootstrap2. 
So if you're project uses bootstrap3, foundation, or some other
css framework, you may want to have a custom base template for the
harvest pages.

I'll omit some of the non-related parts, but an essentially complete
real world example is below. This is from an actual project so just
remember that the naming conventions will be different for you. 'ktb',
'opal', and things are just names of my app.

    {% extends "ktb/opal_base.html" %}
    
    {% block head %}
    <meta charset="utf-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1" />
    <meta name="viewport" content="width=device-width" />
    <link rel=stylesheet href="{{ STATIC_URL }}cilantro/css/bootstrap.css" />
    <link rel=stylesheet href="{{ STATIC_URL }}cilantro/css/bootstrap-responsive.css" />
    <link rel=stylesheet href="{{ STATIC_URL }}cilantro/css/font-awesome.css" />
    <link rel=stylesheet href="{{ STATIC_URL }}cilantro/css/style.css" />
    {% endblock %}
    
    {% block content %}
    <div class="container-fluid">
        <div class="row-fluid">
            <div class="span12">
                <div id="content">
                    <h1>Hello Harvest</h1>
                </div>
            </div>
        </div>
    </div>
    
    <script>
    var require = {
        baseUrl: '{{ STATIC_URL }}cilantro/js',
    },
    
    cilantro = {
        url: '/ktb/serrano/',
        main: '#content',
        root: '/ktb'
    };
    </script>
            
    <script data-main="cilantro/main" src="{{ STATIC_URL }}cilantro/js/require.js"></script>
    {% endblock %}

Some important notes.

* The #content on the main option matches the div ID above. Change one and 
  you need to change the other.
* The url is the serrano API we installed in urls.py previously. In my case the urls.py
  was included by another urls.py that added a 'ktb' prefix.  Just make sure you can load
  the relative URL you see in here and get the RESTful method listing from serrano.
* The root I _think_ is the url root everything will be mounted at.  This includes your
  query, results, and workspaces urls. In my case they actually show up at /ktb/query,
  /ktb/results, and /ktb/workspaces.
* Obviously, just make sure you can load all the cilantro css and require.js at the
  urls you're putting in the template.

Also, don't forget to make a simple view or something for these urls and template.
Something like:

    def harvest(request):
        return render(request, 'ktb/harvest.html', {})


## Add database tables.

Next use south's migrate to add the avocado and serrano tables.

    python manage.py migrate

## Add Avocado metadata

Avocado comes with a command to add some initial metadata for your app.

    python manage.py avocado init appname

TODO more desc on what this actually does and to leave off prefix's if your
app has dots in the name

## Add default view, context, and category

## The future  

Navigate to your query url!
