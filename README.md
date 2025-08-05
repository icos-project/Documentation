# Documentation

This repository cotnains the source code for the **ICOS Techcnical Documentation**. The documentation contains four different guides for the ICOS Meta OS:

- **Concepts**: provides a theoretical introduction to the ICOS Meta OS introducing the architecture, the main functionalities, and the main components. This guide is intended for persons that want to be introduced to ICOS, what it is and how it works.

- **Administration**: provides tutorials and instructions on how to create and manage an ICOS System. This guide is intended for the administrators of an entire ICOS System or a part of it (e.g., an ICOS Agent).

- **User**: provides tutorials and instructions on how to use ICOS Meta OS for running user's applications. This guide is intended for application developers and integrators that want to use ICOS for orchestrating their applications. 

- **Developer**: provides tutorials and instructions on how to use and configure single ICOS components and how to programmatically integrate and/or extend them. It includes a reference for all commands and APIs exposed by ICOS. This guide is intended for expert technical persons who want to learn how ICOS works internally or want to develop integrations or extensions for ICOS components.


## Build

The documentation uses the [Material for MKDocs](https://squidfunk.github.io/mkdocs-material/) framework based on [MKDocs](https://www.mkdocs.org/).

To build a static html website from the source:

```bash
mkdocs build
```

or using Docker as described in the [how to make a change offline](https://production.eng.it/gitlab/icos/documentation/-/wikis/home#make-change-offline)


## Contribution
To contribute please refer to [Wiki Documentation](https://production.eng.it/gitlab/icos/documentation/-/wikis/home)

## Hosting

### Development Version
The development version (corresponding to the `'develop'` branch) is hosted on a [internal ICOS web server](http://10.160.3.154).
The instance is protected, only ICOS partners can be access.
### Stable Version

The stable version of the website (corresponiding to the `'main'` branch) is published in the ICOS Project Website at https://www.icos-project.eu/docs/

