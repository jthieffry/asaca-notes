# Key Notes

## Intro to Containers

* Container images are made of layers. They are all read-only. Modifications create new layers.
* Running containers have the same layers as the corresponding container image PLUS a RW layer, where runtime data is localized.
* Dockerfiles are used to build images.
* Containers are portable and self-contained. Always run as expected.
* They are lightweight. The OS is the parent OS, they are only processes and fs layers are shared.
* Ports are exposed to the host and beyond
* Provide the isolation and only contains what the application needs.