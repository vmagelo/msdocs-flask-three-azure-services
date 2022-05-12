# Deploy a Python (Flask) web app with PostgreSQL and Blob Storage in Azure

This is a Python web app using the Flask framework with three Azure services: Azure App Service, Azure Database for PostgreSQL relational database service, and Azure Blob Storage. This app is designed to be run locally and then deployed to Azure. 

| Function      | Local Dev | Azure Hosted |
| ------------- | --------- | ------------ |
| Web app | runs locally, e.g., http://127.0.0.1:8000 | runs in App Service, e.g., https://\<app-name>.azurewebsites.net  |
| Database | Local PostgreSQL instance | Azure PostgreSQL service |
| Storage | Azure Blob Storage<sup>1</sup> or local emulator like [Azurite emulator for local Azure storage development](https://docs.microsoft.com/en-us/azure/storage/common/storage-use-azurite) | Azure Blob Storage |

A Django sample application is also available for the article at https://github.com/vmagelo/msdocs-django-three-azure-services.

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

Create a virtual environment.

```dos
py -m venv .venv
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
