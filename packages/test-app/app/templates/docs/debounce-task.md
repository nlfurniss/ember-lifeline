# `debounceTask`

**TL;DR - Call `debounceTask(obj, methodName, args*, wait, immediate)` on any object to debounce work.**

Debouncing is a common async pattern often used to manage user input. When a
task is debounced with a timeout of 100ms, it first schedules the work for
100ms later. Then if the same task is debounced again with (again) a timeout
of 100ms, the first timer is cancelled and a new one made for 100ms after
the second debounce request. If no request to debounce that task is made
for 100ms, the task executes.

Here is a good blog post about debounce and throttle patterns:
[jQuery throttle / debounce: Sometimes, less is more!](http://benalman.com/projects/jquery-throttle-debounce-plugin/)

Debouncing is a pattern for managing scheduled work over time, and so it
falls prey to some of the same faults as `setTimeout`. Again Ember provides
`Ember.run.debounce` to handle the runloop aspect, but does not provide
a simple solution for cancelling work when the object is destroyed.

Enter `debounceTask`. For example, no matter how quickly you click on this
component, it will only report the time if you have stopped clicking
for 500ms:

```js
import Component from '@glimmer/component';
import { tracked } from '@glimmer/tracking';
import { debounceTask } from 'ember-lifeline';

export default class Example extends Component {
  @tracked time;

  click() {
    debounceTask(this, 'reportTime', 500);
  }

  reportTime() {
    this.time = new Date();
  }
}
```

However if the component is destroyed, any pending debounce task will be
cancelled.
