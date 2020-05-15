# 2. A YAML manifest’s structure consists of four mandatory elements:

* apiVersion
* kind
* metadata
* spec

The YAML manifest has a hierarchical structure. Each child element is indented (usually by two spaces) from its parent. You can see this under the metadata and spec sections.

In this lab, you create a simple YAML manifest for a Deployment that is very similar to the YAML manifest from the first lab in this module. This time you create it step by step. In each step, you examine your options and determine where you can find the information for your manifest.

### 1. Create a new project called $GUID-yaml-manifests:
`oc new-project $GUID-yaml-manifests --display-name="YAML Manifests"`

## 2.1 Specify API Version and Resouce Kind


