<!-- @@@page:manual@@@ -->
<!-- @@@title:Basic Widgets@@@ -->

Basic Widgets
=============

| Module | Namespace | Stability |
|:-------|:----------|:----------|
| `npm install giant-basic-widgets` | `$basicWidgets` | **Fairly stable** |

GiantJS promotes *component-oriented* architecture. Components are not restricted to the UI, (ie. ~~model-components~~, ~~transport-components~~) but the UI is where they manifest in the most obvious ways. (For a general explanation of components, see ~~About components~~.)

Being component-oriented means **you're not thinking in terms of markup** when you design and code your application. Rater, you start with atomic, *basic* components (widgets), and use them as building blocks to form more complex components, an artifact of which is HTML markup.

This page walks you through GiantJS' official range of atomic widgets, and lets you play around with them.

- Widget events are captured and logged onto the console.
- Hints for each widget class give you ideas about interacting with them programmatically.

** A detailed description of each widget class / variant is to follow. **

<div id="app"></div>
<script src="lib/giant-assertion.js"></script>
<script src="lib/giant-oop.js"></script>
<script src="lib/giant-utils.js"></script>
<script src="lib/giant-data.js"></script>
<script src="lib/giant-event.js"></script>
<script src="lib/giant-routing.js"></script>
<script src="lib/giant-entity.js"></script>
<script src="lib/giant-templating.js"></script>
<script src="lib/giant-i18n.js"></script>
<script src="lib/jquery.js"></script>
<script src="lib/giant-widget.js"></script>
<script src="lib/giant-basic-widgets.js"></script>
<script src="lib/giant-basic-widgets-demo.js"></script>
<link rel="stylesheet" href="lib/giant-basic-widgets.css"/>
<link rel="stylesheet" href="lib/giant-basic-widgets-demo.css"/>
