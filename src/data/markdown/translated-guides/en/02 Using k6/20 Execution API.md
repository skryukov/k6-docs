---
title: Execution API
excerpt: 'get execution information'
---

k6 v0.34 introduces the capability for getting information around the execution process from the test's script.

It can be used importing the `k6/execution` module.

```
import exec from 'k6/execution';

var options = {
	scenarios: {
		'myscenario': {
			executor: 'shared-iterations',
			maxDuration: 30m
		}
	}
}

export default function() {
	console.log(exec.scenario.name) // myscenario 
}
```

The module exports some main root objects: `scenario`, `instance` and `vu`. They can be accessed for getting the specific properties containing the detailed execution information for their specific context.

### Scenario

| Field               | Type    | Description                                                              |
|---------------------|---------|--------------------------------------------------------------------------|
| name                | string  | It is the assigned name of the running scenario.                                |
| executor            | string  | It is the name of the running [Executor](https://k6.io/docs/using-k6/scenarios/#executors).                                                |
| startTime           | integer | It reperesents the unix timestamp in milliseconds when the scenario started.                                                                           |
| progress            | float   | Percentage in a 0 to 1 interval of the scenario progress.                    |
| iterationInInstance | integer | The unique identifier of the single iteration across a single running instance. |
| iterationInTest     | integer | The unique identifier of the single iteration across the entire test. It is unique both in local, in cloud and in distributed or segmented executions. |

### Instance

| Field               | Type    | Description                                                              |
|---------------------|---------|--------------------------------------------------------------------------|
| iterationsInterrupted                | integer  | The amount of iterations interrupted.                                |
| iterationsCompleted     | integer | The amount of iterations completed.  |
| vusActive            | integer  | The amount of VUs active.                                                |
| vusInitialized           | integer | The amount of VUs initialized from the start of the test.                                                                           |
| currentTestRunDuration            | float   | The time passed from the start of the current test run in milliseconds.                    |

### VU

| Field               | Type    | Description                                                              |
|---------------------|---------|--------------------------------------------------------------------------|
| iterationInInstance | integer | The identifier of the iteration in the current instance.                                                                 |
| iterationInScenario | integer | The identifier of the iteration in the current scenario.                  |
| idInInstance        | integer | The identifier of the VU across the instance.                            |
| idInTest            | integer | The global-unique identifier of the VU.                                  |

### Get unqiue data once

One of the common cases,  it's getting an unqiue and incremental index for accessing a dataset's item only once time per test.

The `scenario.iterationInTest` field can be used for this case. Let's show an example:

```
import exec from "k6/execution";
import {SharedArray} from "k6/data";

const data = new SharedArray(); // populate it with a real dataset

export const options = {
    scenarios: {
        "use-all-the-data": {
            executor: "shared-iterations",
            vus: 100,
            iterations: data.length, // here you can also multiply so it goes through it multiple times
            maxDuration: "1h" // this will need to be big enough so that all the iterations can happen if that is what is wanted
        }
    }
}

export default function() {
    var item = data[exec.scenario.iterationInTest];
	// do something with the got item ...
}
```

### Timing things
The `startTime` property from `scenario` object can be used from timing things.

```
import exec from 'k6/execution';

export default function() {
    // do some long operations
	...	
	console.log("step1 took:", new Date() - new Date(exec.scenario.startTime))
	
	// some more long operations
	...
	console.log("step2 took:", new Date() - new Date(exec.scenario.startTime))
}
```

### Script naming
The `name` can be used for executing logics based on which script is currently running.

// TODO: is this a real use case? Should we suggest it or is it a bad practice?

```
import exec from 'k6/execution';

var options = {
	scenarios: {
		'the-first': {
			...
		},
		'the-second': {
			...
		}
	}
};

export default function() {
    if (exec.scenario.name == "the-first") {
		// do some logic during this scenario
	} else {
		// do some other logic in the others
	}
}
```
