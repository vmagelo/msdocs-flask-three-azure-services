# Deploy a Python (Flask) web app with PostgreSQL and Blob Storage in Azure

This is a Python web app using the Flask framework with three Azure services: Azure App Service, Azure Database for PostgreSQL relational database service, and Azure Blob Storage. This app is designed to be run locally and then deployed to Azure. Related: [Django version](https://github.com/vmagelo/msdocs-django-three-azure-services).

| Function      | Local Dev | Azure Hosted |
| ------------- | --------- | ------------ |
| Web app | runs locally, e.g., http://127.0.0.1:8000 | runs in App Service, e.g., https://\<app-name>.azurewebsites.net  |
| Database | Local PostgreSQL instance | Azure PostgreSQL service |
| Storage | Azure Blob Storage<sup>1</sup> or local emulator like [Azurite emulator for local Azure storage development](https://docs.microsoft.com/en-us/azure/storage/common/storage-use-azurite) | Azure Blob Storage |

<sup>1</sup>Current code can work with Azure Blob Storage accessed from local environment or Azurite (local storage emulator). The same environment variables STORAGE_ACCOUNT_NAME and STORAGE_CONTAINER_NAME in *.env* are used for both cases. The code figures out what to do for each case. And AzureDefaultCredential is used in both cases. Note: to use Azure, we need to run Django with SSL, which would require a certificate and adding some libraries. See [Tip 1](#tip-1-using-ssl) for using SSL. See [Tip 2](#tip-2-using-azurite) for using Azurite.


## Todo

1. Swap out psycopg2 for psycopg2-binary.

## Deployment

Create all Azure resources in the same group.

1. Set up App Service.
    * Set managed identity following [Auth from Azure-hosted apps](https://review.docs.microsoft.com/en-us/azure/developer/python/sdk/authentication-azure-hosted-apps)
    * Set managed identity as system-assigned
    * Assign role as "Storage Blob Data Contributor", so app service can connect to storage

1. Set up PostgreSQL
    * Add firewall rule so local machine can connect (necessary if you are creating table in VS Code, otherwise optional)
    * "Allow public access from any Azure service" as we did in previous tutorial. **Can we use managed identity?** See [Tip 8](#tip-8---postgresql).

1. Set up Azure Storage.
    * Create container "photos".
    * Set container to "public read". This doesn't mean public write. The web app can write because of the assigned role.

1. Deploy the app with one of the methods: VS Code, local git, ZIP.
    * set app service configuration variables for: DBNAME, DBHOST, DBUSER, DBPASS, STORAGE_ACCOUNT_NAME, STORAGE_CONTAINER_NAME
    * ssh into app service
    * The deployment causes the database schema to be created. If they aren't in the SSH, try `flask db init`.

## Requirements

The [requirements.txt](./requirements.txt) has the following packages:

| Package | Description |
| ------- | ----------- |
| [Flask](https://pypi.org/project/Flask/) | Web application framework. |
| [SQLAlchemy](https://pypi.org/project/SQLAlchemy/) | Provides a database abstraction layer to communicate with PostgreSQL. |
| [Flask-SQLAlchemy](https://pypi.org/project/Flask-SQLAlchemy/) | Adds SQLAlchemy support to Flask application by simplifying using SQLAlchemy. Requires SQLAlchemy. |
| [Flask-Migrate](https://pypi.org/project/Flask-Migrate/) | SQLAlchemy database migrations for Flask applications using Alembic. Allows functionality parity with Django version of this sample app.|
| [pyscopg2](https://pypi.org/project/psycopg2/) | PostgreSQL database adapter for Python. |
| [python-dotenv](https://pypi.org/project/python-dotenv/) | Read key-value pairs from .env file and set them as environment variables. In this sample app, those variables describe how to connect to the database locally. <br><br> Flask's [dotenv support](https://flask.palletsprojects.com/en/2.1.x/cli/#environment-variables-from-dotenv) sets environment variables automatically from an `.env` file. |
| [flask_wtf](https://pypi.org/project/Flask-WTF/) | Form rendering, validation, and CSRF protection for Flask with WTForms. Uses CSRFProtect extension. |
| [azure-blob-storage](https://pypi.org/project/azure-storage/) | Microsoft Azure Storage SDK for Python |
| [azure-identity](https://pypi.org/project/azure-identity/) | Microsoft Azure Identity Library for Python |

## How to run (Windows)

Create a virtual environment. For SSL work, 3.9 version is best, especially with python-certifi-win32 package.

```dos
py -3.9 -m venv .venv
.venv/scripts/activate
```

In the virtual environment, install the dependencies.

```dos
pip install -r requirements.txt
```

Create the `restaurant` and `review` tables.

```dos
flask db init
flask db migrate -m "initial migration"
```

Run the app.

```dos
flask run
```
## Tips and tricks learned during development

See the tips and tricks of the [Django version](https://github.com/vmagelo/msdocs-django-three-azure-services) of this app. Many of the points there apply here except where noted.

### Tip 1: Using SSL

Flask is a little easier to get running with SSL than Django. One less step. 

The steps for getting SSL to run are roughly this:

Step 1: Install [mkcert](https://github.com/FiloSottile/mkcert). Run it and create a certificate. The alternative if you don't use it is that your browser will keep showing warning.

```
mkcert --install
mkcert -cert-file cert.pem -key-file key.pem localhost 127.0.0.1 
```

Step 2: Use machine CA certificates. (Only do this with Python 3.9 or perhaps earlier. Did not work in Python 3.10)

```
pip install python-certifi-win32
```

Step 3: Change the run command to:

```
 flask run --cert=cert.pem --key=key.pem
```

### Tip 2: Using Azurite

We recommend using [Azurite](https://docs.microsoft.com/en-us/azure/storage/common/storage-use-azurite) from the command line to emulate blob storage that can be used by the web app. (Using it from Visual Studio Code with an extension is easier, but couldn't get past SSL problems. Plus, running from command line is more general and can potentially reach more users.)

```dos
azurite-blob ^
    --location "<folder-path>" ^
    --debug "<folder-path>\debug.log" ^
    --oauth basic ^
    --cert "<project-root>\cert.pem" ^
    --key "<project-root\key.pem"
```

### Tip 3: Passing error messages 

In the Django version of this app, we used the built in Django [messages framework](https://docs.djangoproject.com/en/4.0/ref/contrib/messages/). That doesn't exist in Flask. There are a couple of options:

* Use [Flask-Session package](https://pypi.org/project/Flask-Session/) to add server-side session.
* Use normal session object already present in Flask. It uses cookies.
* Don't use any session and use a dedicated variable passed to template.

We went for the last option. If there is no message just set message to empty string.

### Tip 4: Flask auto-reload

[ref](https://stackoverflow.com/questions/16344756/auto-reloading-python-flask-app-upon-code-changes)

```bash
export FLASK_APP=app.py
export FLASK_ENV=development
flask run
```

```dos
set FLASK_APP=app.py
set FLASK_ENV=development
flask run
```




