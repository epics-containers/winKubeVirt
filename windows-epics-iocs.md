# How Windows IOCs will fit into epics-containers

## Overview

Windows IOCs are a last resort for when the manufacturer only supplies drivers on the Windows platform. This will typically be for detectors and each detector would typically have its own dedicated windows server for running the vendor software and an IOC.

A windows build server is required this should be set up using one of the methods described here https://docs.epics-controls.org/en/latest/getting-started/installation-windows.html.

The build server will also require the SDK from the relevant manufacturer.