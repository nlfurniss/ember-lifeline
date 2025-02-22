# `scheduleTask`

**TL;DR - Call `scheduleTask(obj, queueName, fnOrMethodName, args*)` on any object to schedule work on the run loop.**

Use `scheduleTask` where you might use `Ember.run.schedule`.

Like `runTask`, `scheduleTask` avoids common pitfalls of deferred work.

_`Ember.run.schedule` does not bind the scheduled work to the lifecycle of the context object_.

```js
import Component from '@glimmer/component';
import { tracked } from '@glimmer/tracking';
import { run } from '@ember/runloop';

export default class Example extends Component {
  @tracked date;

  constructor() {
    super(...arguments);

    run.schedule('actions', this, () => {
      this.date = new Date();
    });
  }
}
```

There's a chance that objects scheduling work may be destroyed by the time the
queue is flushed. Leaving behavior to chance invites flakiness. This manifests
as a number of unexpected errors. Fixing this issue requires checks for
`isDestroyed` state on objects retained after destruction:

```js
import Component from '@glimmer/component';
import { tracked } from '@glimmer/tracking';
import { run } from '@ember/runloop';

export default class Example extends Component {
  @tracked date;

  constructor() {
    super(...arguments);

    run.schedule('actions', this, () => {
      // First, check if this object is even valid
      if (this.isDestroyed) {
        return;
      }
      this.date = new Date();
    });
  }
}
```

The code above is correct, but less than ideal. Instead, always use
`scheduleTask`. `scheduleTask` entangles a scheduled task with the lifecycle of
the object scheduling the work. When the object is destroyed, the task is also
cancelled.

Using `scheduleTask`, the above can be written as:

```js
import Component from '@glimmer/component';
import { tracked } from '@glimmer/tracking';
import { scheduleTask } from 'ember-lifeline';

export default class Example extends Component {
  @tracked date;

  constructor() {
    super(...arguments);

    scheduleTask(this, 'actions', () => {
      this.date = new Date();
    });
  }
}
```

#### A word about the `afterRender` queue

Scheduling work on the `afterRender` queue has well known, negative performance implications.
Therefore, _`scheduleTask` is prohibited from scheduling work on the `afterRender` queue._
