<!-- @@@page:manual@@@ -->
<!-- @@@title:Introduction@@@ -->

Introduction
============

Giant is a full-featured, modular framework for building complex, modern Single-Page Applications. It targets multi-platform, enterprise-grade projects, stimulating unlimited, steady growth.

The purpose of Giant is to give developers a *single, coherent system* with recurring practices and patterns, serving as a solid foundation to a *wide variety* of applications. At the same time, Giant's modular structure allows for a great deal of flexibility, as any of its modules may be replaced with a functionally equivalent (set of) libraries. Or, individual modules of Giant may be used to fill the gaps in other frameworks.

Parts of Giant have been used in various projects since 2010. It's been revised, improved, and for some parts, redesigned multiple times, based on experience building actual, ambitious, large-scale applications. It's a framework which, despite being 'new', is past its maturation phase.

Directives
----------

Giant is governed by five directives, which ensure forward compatibility and guide the development process even when implementation details change.

1. **Modularity**: Keeping it a family of *independent* but *interconnected* libraries.
2. **Extensibility**: Allowing framework components to be augmented or changed.
3. **Closed dependency tree**: Limiting external dependencies to a minimum.
4. **Feature richness**: Covering all aspects of modern single page applications.
5. **Recurring patterns**: Applying the same practices to similar tasks throughout the framework and application.

Giant encourages the application of these directives to the SPAs built with it.

Modules
-------

In Giant, each module takes care of a specific aspect of the application. Modules are self-containing [npm](http://npmjs.org) packages, able to run both in modern browsers and under [Node.js](https://nodejs.org). On the figure below, each bubble represents a module of the framework, organized into 3 tiers: *essentials*, *core*, and *auxiliary*. Modules in one tier may only depend on modules in the same or lower tiers. Modules that are grayed out are not published yet.

![Module structure (Open in new tab to magnify)](https://raw.githubusercontent.com/giantjs/giant-developer-guide/master/images/Giant%20Modules.png)

### Essentials tier

Basic functionality for building classes, logging errors, coupling components, and managing templates and data objects.

- **assertion**: Provides a few built-in assertions and an interface to add new ones.
- **oop**: Implements a powerful OOP toolkit that allows multiple inheritance, instance caching, and reflection; provides means for performance tuning and unit testing.
- **utils**: Low-level utilities, eg. async tools, promises, as well as string, and array manipulation. 
- **data**: Hash-based data structures such as collections, dictionaries, sets, trees, with functional APIs.
- **event**: Event dispatching and subscription mechanism that is universally applicable to anything that may be represented by a path. Allows events to be traced back to their origin.
- **templating**: Flexible, evented string-replacement.
- **ajax**: Evented API around URLs and `XMLHttpRequest`.
- **table**: Hash-based implementation of querying and updates on indexed JSON objects that represent tables.

### Core tier

Low-level and common application components. The core tier alone is enough to build applications, when auxiliary tier models are not necessary or not suitable.

- **rest (transport)**: Implements evented access to REST APIs. Decouples AJAX requests from application components consuming API responses.
- **entity**: Introduces an evented in-memory datastore for managing documents, fields, and collections, as well as lookups and search indexes. 
- **routing**: Provides an API for navigation and notifies on route changes.
- **widget**: Basis for stateful view-controllers. Widgets are self-contained, arranged in a high-level hierarchy (approximating the DOM), and have a life cycle.
- **i18n**: Provides API for switching locale & obtaining localized strings. Maintains evented, application-wide locale state, introduces quasi-strings that serialize according to the current locale. JSON-based, compatible with [gettext](https://www.gnu.org/software/gettext/). Agnostic re. JSON source.
- **session**: Provides API for user authentication. Maintains evented, application-wide state for current user. Back-end agnostic.
- **basic-widgets (common-widgets)**: Set of commonly used skinless widgets, based on the **widget** module.

### Auxiliary tier

High-level components for assisting development, deployment, and logging.

- **asset**: Manages application assets, both at build / deployment, as well as loading.
- **cli-tools**: Implements classes for dealing with CLI arguments in a uniform way.
- **diagnostic-widgets**: A set of skinned widgets providing insight to the developer about current application state.
- **ga-tools**: Provides high-level API for connecting the application with [Google Analytics](https://www.google.com/analytics/).
- **grunt-tools**: Provides high-level and uniform API for dealing with [Grunt](http://gruntjs.com/) tasks.
- **material-widgets**: Adds off-the-shelf [Google Material Design]() widgets on top of **widgets** and **basic-widgets**.
- **popup**: Adds off-the-shelf popup management for dealing with stacking and queueing.
- **automation**: Introduces tools for automating visual regression testing.
- **cli**: Essential command line tools for building Giant-based applications, initializing projects and application modules.

Application architecture
------------------------

The typical Giant-based application is made up of a handful of high-level components. Each component serves a specific purpose, and is based on one or more of the framework modules.

In the figure below, each block represents one such component, using and listening in on connected components.

![Architecture](https://raw.githubusercontent.com/giantjs/giant-developer-guide/master/images/Architecture.png)

### General

Serves the entire front end stack. Based on *essentials-tier* modules, such as `giant-assertion`, `giant-oop`, `giant-utils`, `giant-data`, `giant-events`, and `giant-templating`, as well as application-domain general utilities.

### Transport

Communicates with the server. Based on `giant-ajax` and parts of `giant-rest`. Controls and listens to the browser's `XMLHttpRequest` object.

### Model

Manages the application's data layer. Based on `giant-entity`, parts of `giant-rest` and optionally `giant-table`.

- Provides access and manipulation to entities
- Maintains lookups and indexes
- Listens to whether any data is required from the server
- Integrates server responses into the entity store, and through it, the rest of the application state
- Notifies relevant components of entity changes

Controls and listens to the 'transport' component.

### Widgets

Making up most of an application's codebase, this component is where UI logic is implemented. In an MVC approach, the 'widgets' component is equivalent to the view-controller. Based on `giant-widget`, `giant-basic-widgets`, and other widget modules.

- Generates and injects markup into the DOM, based on current application state
- Synchronizes the UI state and DOM to other application components, primarily the model 

Controls and listens to the DOM, 'model', 'session', 'i18n', 'routing', and 'general' components. It is not allowed to access the 'transport' component directly though.

### Session

Manages the authenticated user. Based on `giant-session`.

- Handles user authentication
- Maintains & persists session information
- Notifies relevant components of session-related changes

Controls and listens to the 'model' and 'transport' modules, and is being controlled by the 'widgets' component.

### Internationalization

Manages locales and translated text. Based on `giant-i18n`.

- Maintains current locale
- Provides API to accessing text according to current locale
- Merges translations and other locale information into the model
- Notifies relevant components of locale-related changes

Controls and listens to the 'model' component, is being controlled by 'widgets'. 

### Routing

Manages navigation within the application. Based on `giant-routing`.

- Maintains current route
- Provides navigation API
- Notifies relevant components of route changes

Controls `window.location`, is controlled by the 'widgets' component.
