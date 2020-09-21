---
layout: home
title: Curiefense Documentation
permalink: /documentation/
sidebar:
  nav: "documentation"
toc: true
toc_label: "On this page"
toc_icon: "cog"
---

![Curiefense logo](/assets/images/9_open_icon.png "Curiefense logo"){: style="padding:10px; width:35%; height:35%;"}

Curiefense is an API-first, DevOps oriented web-defense HTTP-Filter adapter for Envoy. It provides next-generation WAF capabilities and real-time traffic monitoring and transparency.

Curiefense is fully controllable programmatically. All configuration data (security rulesets, policies, etc.) can be maintained singularly or as different branches for different environments, as you choose. All changes are versioned, and reverts can be done at any time.

Curiefense also has a UI console, discussed in this Manual in the Console and Analytics sections. 

If you are unfamiliar with Curiefense, please start by reading the Architecture and Overview.

---

# Deployment Options

Curiefense can run in variety of environments, depending on your specific needs. It can be adapted to many different use cases. 

Deployment and installation instructions for several different environments are available in the Installation section of this manual. More will be added in the future.

If you create an installation workflow for a situation that is not currently described in this manual, please feel free to submit it for inclusion.

---

# Architecture

Curiefense provides traffic filtering that can be configured differently for multiple environments, all of which can be administered from one central cluster if desired. Here is an overview of its components.

![Curiefense architecture](/documentation/images/Architecture.png "Curiefense architecture")

Here's a more detailed view:

![Curiefense details](/documentation/images/Architecture-details.png "Curiefense details")

In this diagram, a single cluster is shown. Multiple-cluster architectures are also possible.

---

# Three Primary Roles

Conceptually, there are three primary roles performed by Curiefense:

- **Configuration** (allowing admins to define security policies, assign them to URLs, etc.)

- **Filtering** (applying the defined Configurations to incoming traffic and blocking hostile requests)

- **Monitoring** (displaying traffic data in real-time and in historical logs).

---

# Configuration

### Data Structures

Curiefense maintains its security parameters as Entries, which are contained in Documents, which are contained in Configurations.

A Configuration is a complete definition of Curiefense's behavior for a specific environment. An organization can maintain multiple Configurations (e.g., development, staging, and production).

![Curiefense data structure](/documentation/images/Data-structure.png "Curiefense data structure")

Each Configuration contains six Documents (one of each type: ACL Profiles, Rate Limits, etc.) Each Document contains at least one Entry, i.e., an individual security rule or definition. Documents are edited and managed in the Document Editor.

A Configuration also includes blobs, which currently are used to store the Maxmind geolocation database. This is where Curiefense obtains its geolocation data and ASN for each request it processes.

# Configuration Administration

A Configuration is the atomic unit for all of Curiefense's parameters. Any edits to a Configuration result in a new Configuration being committed. Configurations are versioned, and can be reverted at any time.

The second architectural diagram above shows the information flow for a single-cluster architecture. On the right, there are two containers (**confserver** and **uiserver**) which provide different means of editing Configurations:

- The **confserver** provides a REST API, and stores a complete history of Configurations in a database (**Conf Db**, which by default is a git repository). The **API Client** enables the editing of Configurations using cURL and/or the Curiefense CLI tool. 

- The **uiserver** provides a graphic UI for editing Configurations (and also for presenting traffic data to the user through the Access Log and other means).
When a Configuration is created or modified, the admin pushes it to a **Cloud Storage** bucket. The **curiesync** container monitors its specified bucket(s); when a change is pushed to the bucket(s), **curiesync** will update the **curiefense** Envoy plugins.

An important feature of Curiefense is simultaneous publishing to multiple environments. 

![Curiefense and multiple environments](/documentation/images/Architecture-Multiple-buckets.png "Curiefense and multiple environments")

When a Configuration is published, it can be pushed to multiple buckets (each of which can be monitored by one or more environments) all at once, from a single button-push or API call.

---

# Filtering

In the architectural diagram, **curiefense** (shown on the bottom left) performs the actual traffic filtering. In other words, this is where the security policies defined in the Configurations are enforced.

Internally, **curiefense** uses Redis for rate limiting and aggregated metrics. Other storage methods can be used instead if desired.

---

# Monitoring

Each time a request goes through **curiefense**, a detailed log message is pushed to the **curielogger** container, which sends it to the **logdb**. The **logdb** saves the data to the specified database. (Out of the box, Curiefense is setup to work with postgreSQL, but any RDBMS can be used.)

Traffic data is available in several ways:

- The Curiefense graphical client provides an Access Log which provides comprehensive details for requests.

- Curiefense is also integrated with Grafana and Prometheus, for traffic dashboards and other displays.
