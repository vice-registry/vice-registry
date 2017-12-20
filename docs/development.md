# ViCE-Registry Documentation

## Structure

This documentation consists of several sections:

1. [Motivation & Use Cases](./motivation.md)
2. [Installation  Guide](./installation.md)
3. [Usage Guide](./usage.md)
4. [Development Guide](./development.md)

## Development Guide

The ViCE-Registry components can be found in their own Github repositories:

Build State for each component:
 * [ViCE API](https://github.com/vice-registry/vice-api) [![Build Status](https://travis-ci.org/vice-registry/vice-api.svg?branch=master)](https://travis-ci.org/vice-registry/vice-api)
 * [ViCE Worker](https://github.com/vice-registry/vice-worker)  [![Build Status](https://travis-ci.org/vice-registry/vice-worker.svg?branch=master)](https://travis-ci.org/vice-registry/vice-worker)
 * [ViCE Store](https://github.com/vice-registry/vice-store) [![Build Status](https://travis-ci.org/vice-registry/vice-store.svg?branch=master)](https://travis-ci.org/vice-registry/vice-store)
 * [ViCE WebUI](https://github.com/vice-registry/vice-webui) [![Build Status](https://travis-ci.org/vice-registry/vice-webui.svg?branch=master)](https://travis-ci.org/vice-registry/vice-webui)

## Architecture

The basic architecture of the ViCE-Registry looks as follows:
![ViCE Registry Architecture](vice-registry-architecture.png "ViCE Registry Architecture")

These layers are implemented by components, which are micro services communicating
over RabbitMQ as message queue:
![ViCE Registry Components](vice-registry-components.png "ViCE Registry Components")
