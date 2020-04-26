

## Directory structure

Create a venv at the same level as this project folder. Also, create a static
files directory at the this same level for STATIC_ROOT.

## Using django-pipeline to minify and cache-bust static files

I believe this site is configured correctly to run django-pipline.  Remember these key points:

1. If DEBUG = True, this means you're in "development mode" and you should be running Django's development server. Then run 'collectstatic' and Django will copy all the static files it knows about to STATIC_ROOT and serve them for you.

2. If DEBUG = False and you're running Django's web server, Django will not serve the static files. Your site will work but you will get 404 errors for all linked static files. In this situation, you should be running Gunicorn as your webserver behind Nginx, and Nginx must be configured to serve the static files. Setting up Gunicorn and Nginx are a pain on macOS so I'm not going to attempt it.

    NOTE: If you attempt to run Gunicorn and Nginx with DEBUG = True (i.e. in development mode), you'll need to modify 'urlpatterns' in your urls.py file:

        from django.contrib.staticfiles.urls import staticfiles_urlpatterns
        # ... the rest of your URLconf goes here ...
        urlpatterns += staticfiles_urlpatterns()

    See [How to make Django serve static files with Gunicorn?](https://stackoverflow.com/questions/12800862/how-to-make-django-serve-static-files-with-gunicorn)

## Using django-pipeline template tags

Assume this PIPELINE setting:

    PIPELINE = {
        # Default: not settings.DEBUG
        #'PIPELINE_ENABLED': True,
        'STYLESHEETS': {
            'basecss': {
                'source_filenames': (
                    'app2/app2.css',
                    'app1/app1.css',
                ),
                'output_filename': 'css/styles.css',
            },
        },
        'JAVASCRIPT': {
            'basejs': {
                'source_filenames': (
                    'app1/app1.js',
                    'app2/app2.js',
                ),
                'output_filename': 'js/base.js',
            }
        }
    }

To include the above stylesheets in your template, you would specify the stylesheet group name 'basecss' at the top of your templates:

    {% stylesheet 'basecss' %}

What happens is this:

If DEBUG = True, each of the stylesheets in 'source_filenames' will be linked into the template and they will be unminified and will not have a GUID.  This is done to make debugging easier.

If DEBUG = False, a 'css/styles.css' stylesheet will be created when you run 'collectstatic' and it will be linked into your templates.  It will be minified, the filename will contain a GUID, and each template will link to this new filename that includes the GUID.  This default behavior can be overrridden.

See django-pipeline [Templatetags](https://django-pipeline.readthedocs.io/en/latest/usage.html#templatetags)

## Running npm to install Yuglify

django-pipeline requires Yuglify.

    # Install npm
    brew update
    brew install node

    # Create package.json file
    npm init --yes

    # Note that this installs yuglify in 'dependencies'.  If you were to
    # specify "--save-dev" it would go in the (development) 'devDependencies'
    # section instead.
    # Note also that if you want to install yugligy globally, you'd use the
    # '--global' argument.

    npm i[nstall] yuglify

    # Check if package installed
    npm search yuglify

    # List all packages
    npm list [ --local | --global ]
