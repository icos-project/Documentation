# Documentation

This repository cotnains the source code for the ICOS Techcnical Documentation. The documentation contains four different guides for the ICOS Meta OS:
- **Concepts Guide**: provides a theoretical introduction to the ICOS Meta OS introducing the architecture, the main functionalities, and the main components. This guide is intended for persons that want to be introduced to ICOS, what it is and how it works.
- **Administration Guide**: provides tutorials and instructions on how to create and manage an ICOS System. This guide is intended for the administrators of an entire ICOS System or a part of it (e.g., an ICOS Agent).
- **User Guide**: provides tutorials and instructions on how to use ICOS Meta OS for running user’s applications. This guide is intended for application developers and integrators that want to use ICOS for orchestrating their applications. 
- **Developer Guide**: provides tutorials and instructions on how to use and configure single ICOS components and how to programmatically integrate and/or extend them. It includes a reference for all commands and APIs exposed by ICOS. This guide is intended for expert technical persons who want to learn how ICOS works internally or want to develop integrations or extensions for ICOS components.


## Build

The documentation uses the [Material for MKDocs](https://squidfunk.github.io/mkdocs-material/) framework based on [MKDocs](https://www.mkdocs.org/).

To build a static html website from the source:

```bash
mkdocs build
```

or using Docker:

```bash
docker run --rm -it -v ${PWD}:/docs squidfunk/mkdocs-material build
```


## Contribution
To contribute please refer to https://production.eng.it/gitlab/icos/documentation/-/wikis/home

## Hosting

### Development Version
The development version of the website (corresponding to the `develop` branch) is published at

        https://docs.dev.icos.91.109.56.214.sslip.io/docs

The Instance is protected by a firewall, ask ENG for allowing access.

### Stable Version

The stable version of the website (corresponiding to the `main` branch) is published in the ICOS Project Website at https://www.icos-project.eu/docs/

