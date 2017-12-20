# ViCE-Registry Documentation

## Structure

This documentation consists of several sections:

1. [Motivation & Use Cases](./motivation.md)
2. [Installation  Guide](./installation.md)
3. [Usage Guide](./usage.md)
4. [Development Guide](./development.md)

## Usage Guide

The ViCE Registry can be used via the WebUI dashboard or via the RESTful API endpoint.

### Web Dashboard

1. The dashboard can connect to any API endpoint. When the dashboard is opened
   for the first time, the API endpoint needs to be configured. A warning
   appears in the top right navigation bar. ![ViCE Registry WebUI
   Configuration](screenshots/configuration.png "ViCE Registry WebUI
   Configuration")

2. When the API endpoint is set, a new user can be registered or an existing
   user can login to use the ViCE registry. ![ViCE Registry WebUI Login
   Register](screenshots/login_register.png "ViCE Registry WebUI Login
   Register")

3. The first step is to register an execution environment. This can be e.g. an
   OpenStack deployment. ![ViCE Registry WebUI
   Environments](screenshots/environment.png "ViCE Registry WebUI Environments")

4. Finally, virtual environments represented as images can be imported into the
   ViCE registry. ![ViCE Registry WebUI Import](screenshots/import.png "ViCE
   Registry WebUI Import")

5. Already imported images can then be exported via deployments to other
   execution environments. ![ViCE Registry WebUI Export](screenshots/deploy.png
   "ViCE Registry WebUI Export")


### REST API

The REST API is defined with [Swagger](https://swagger.io/), which allows to
export clients in any supported programming language. 

[A documentation of available REST calls is available here.](./api/index.html)