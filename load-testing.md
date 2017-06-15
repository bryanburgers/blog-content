At work recently, I needed to load test some of our services. Unfortunately,
the usual load testing tools like [Siege][siege] and [ab][ab] weren't options
because the service under test isn't an HTTP endpoint.

Instead, I needed to simulate thousands or tens of thousands of physical
devices – I suppose you could call them Interent of Things devices – connecting
to a TCP-based service.

After thinking about how to do this, I settled on using EC2 Cluster Service
(ECS) to spin up a cluster, and then run docker images of the thousands of
devices.

Using the AWS API, I could combine [ASG's
SetDesiredCapacity][set-desired-capacity] with [ECS'
DescribeClusters][describe-clusters] to create a script that easily scales the
cluster up and down as needed. Something roughly like:

```js
async function scaleCluster(instances) {
  await setDesiredCapacity(ASG_NAME, instances)
  while (true) {
    const result = await describeClusters([CLUSTER_NAME])
    if (result.clusters[0].registeredContainerInstancesCount === instances) {
      break
    }
    await timeout(1000)
  }
}
```

And then I could use [ECS' RunTask][run-task] to kick off each simulated device
and again use [DescribeClusters][describe-clusters] to wait until the expected
number of tasks had been spun up.

In my situation, I would need to kick off each task individually, because each
device needed its own unique ID, which would be passed as an environment
variable. 

And I certainly didn't want them to all come online at _exactly_ the same time
and slam the server. While that would be one type of load test, it wouldn't
simulate the real world in the way I wanted. So I would need to spread out
spinning up the devices over the course of a few minutes.

Maybe something roughly like:

```js
async function addTasks(taskCount) {
  for (let i = currentCount; i < taskCount; i++) {
    const delayMilliseconds = Math.random(0, 60 * 1000)
    runTask(uuid(), delayMilliseconds)
  }

  while (true) {
    const result = await describeClusters([CLUSTER_NAME])
    if (result.clusters[0].runningTasksCount === taskCount) {
      break
    }
    await timeout(1000)
  }
}

async function runTask(id, delayMilliseconds)
  await timeout(delayMilliseconds)
  await ecs.runTask({
    cluster: CLUSTER_NAME,
    count: 1,
    taskDefinition: TASK_NAME,
    overrides: {
      containerOverrides: [
        {
          environment: [ { name: 'UUID', value: id } ],
          name: CONTAINER_NAME,
        }
      ]
    }
  ).promise()
}
```

Combining these into a script gave me a nice little load-test controller.

<script type="text/javascript" src="https://asciinema.org/a/bxfjcno8lahktd4os8q22v2qm.js" id="asciicast-bxfjcno8lahktd4os8q22v2qm" async></script>


[siege]: https://www.joedog.org/siege-home/
[ab]: https://httpd.apache.org/docs/2.4/programs/ab.html
[describe-clusters]: http://docs.aws.amazon.com/AmazonECS/latest/APIReference/API_DescribeClusters.html
[run-task]: http://docs.aws.amazon.com/AmazonECS/latest/APIReference/API_RunTask.html
[set-desired-capacity]: http://docs.aws.amazon.com/AutoScaling/latest/APIReference/API_SetDesiredCapacity.html
