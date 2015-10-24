Introduction
============

Giant is a full-featured framework for building complex, modern, snappy [single-page applications](https://en.wikipedia.org/wiki/Single-page_application) (SPAs). Giant targets large, enterperise-grade projects, ensuring maintainable, steady growth towards and beyond hundreds of thousands of lines of code.

Giant's modular structure allows for a great deal of flexibility, as any of its modules may be replaced with a functionally equivalent (set of) libraries. Or, individual modules of Giant may be used to fill the gaps in other frameworks. The purpose of Giant however, is to give developers a *single tool* with familiar practices and patterns recurring across modules, serving as a solid foundation to a *wide variety* of applications.

Parts of Giant have been used in various projects since 2010. It's been revised, improved, and for some parts, redesigned multiple times, based on experience building actual, ambitious, large-scale applications. It's a framework which, despite being 'new', is past its maturation phase.

Directives
----------

Giant is governed by five directives that are set to be observed even when implementation details change.

1. **Modularity**: Keeping it a family of *independent* but *interconnected* libraries.
2. **Extensibility**: Allowing framework components to be augmented or changed.
3. **Closed dependency tree**: Limiting external dependencies to a minimum.
4. **Feature richness**: Covering all aspects of modern front end applications.
5. **Recurring patterns**: Applying the same practices to similar tasks throughout the framework and application.

Modules
-------

In Giant, each module takes care of a specific aspect of the application. Modules are self-containing [npm](http://npmjs.org) packages, able to run both in modern browsers and under [Node.js](https://nodejs.org). On the figure below, each bubble represents a module of the framework, organized into 3 tiers: core, framework, and application. Modules in one tier may only depend on modules in the same or lower tiers. Modules that are grayed out are not published yet.

![Module structure](https://raw.githubusercontent.com/giantjs/giant-developer-guide/draft/images/Giant%20Modules.png)

### Core tier modules

The core tier covers basic functionality for building classes, logging errors, coupling components, and managing data objects.

- Extensible assertions
- OOP and testing
- General utilities
- Data structures
- Universal events
- String templating
- Ajax
- Indexed tables

### Framework tier modules:

Low-level and common application components.

- API access
- Entity system
- Routing
- Widget system
- Internationalization
- Session management
- General purpose widgets

### Application tier modules:

High-level components for assisting development, deployment, and logging.

- Asset management
- CLI building tools
- Diagnostic widgets
- Google Analytics tools
- Grunt tools
- Material design widgets
- Popup management
- Test & build automation
- Framework CLI
