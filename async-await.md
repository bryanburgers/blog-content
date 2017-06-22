It's finally happened – `async`/`await` has come to a Node LTS. With the
release of [Node 8][node8] this year, [async and await][asyncawait] are
available in a stable release of Node (meaning one I can convince my employers
to use) without having to resort to transpilation.

While we're still going to let Node 8 bake for a bit at my job before adopting
it wholesale, I've already seen a few ways that `async`/`await` improve code
readability in some of the more annoying areas of asynchronous JavaScript code.

Here are two common coding structures that I run into on occassion that become
much clearer with the introduction of this major new JavaScript language
feature.


## Complex functions

Here is an example of everyone's favorite pyramid-of-doom strawman. With
callbacks, we need to add indentation for every asynchronous call. And in this
particular strawman, it's hard to split up into separate functions because we
need to return the first result after performing all of the other work.

```js
function calculateAandD(callback) {
  calculateA((err, a) => {
    if (err) return callback(err)

    calculateB(a, (err, b) => {
      if (err) return callback(err)

      calculateC(a, b, (err, c) => {
        if (err) return callback(err)

        calculateD(c, (err, d) => {
          if (err) return callback(err)

          return callback(null, { a, d })
        })
      })
    })
  })
}
```

Promises allow us to remove a lot of boilerplate from our callback code, and
even let us drop a level of indentation because we don't need one of the
intermediate values. However, because we need to hold on to the result of the
first promise, we still end up with code that is harder to read and maintain
than we'd like.

```js
function calculateAandD() {
  return calculateA().then(a => {
    return calculateB(a)
      .then(b => calculateC(a, b))
      .then(c => {
        return calculateD(c).then(d => {
          return { a, d }
        })
      })
    })
  })
}
```

But with `async`/`await`, we can finally write this asynchronous code in a
readable, maintainable, and almost synchronous style.

```js
async function calculateAandD() {
  const a = await calculateA()
  const b = await calculateB(a)
  const c = await calculateC(a, b)
  const d = await calculateD(c)

  return { a, d }
}
```

I see this structure come up quite a bit when doing transactional database code
that has to use the inserted ID to update relational tables.

```js
async function createPerson({ name, email, address }) {
  const insertId = await insertIntoPersonTable(name)
  await insertIntoPersonEmailTable(insertId, email)
  if (address) {
    await insertIntoPersonAddressTable(insertId, address)
  }

  return insertId
}
```


## Waiting for a result

Another type of code I had to write recently that `async`/`await` makes
considerably more readable is kicking off a long-running process and then
checking and waiting for it to complete.

In order to accomplish this with promises, we need to use recursion. This might
be readable if you come from a functional-programming background, but even then
it requires creating a new function to do the recusion, and the program
structure doesn't make it very clear what we're actually doing here.

```js
function startAndWait() {
  return startProcess().then(id => {
    function wait() {
      return checkIfComplete(id).then(done => {
        if (done) {
          return id
        }

        return sleep(1000).then(() => wait())
      })
    }

    return wait()
  })
}
```

I just read that code 10 minutes after writing it, and it took some serious
thinking to understand what it does.

With `async`/`await`, we can get rid of the recursion completely and use a
more-readable while loop. This code more clearly maps to what we're doing:
**while** the process isn't finished yet, periodically check if it is done.

```js
async function startAndWait() {
  const id = await startProcess()
  while (true) {
    const done = await checkIfComplete(id)
    if (done) {
      break
    }
    await sleep(1000)
  }

  return id
}
```

An example of this is what I talked about in [my previous post][loadtesting]
when we want to make an API call to AWS and then periodically fetch the state
of of the resource until the provisioning is done.

```js
async function scaleCluster(instances) {
  await setDesiredCapacity(ASG_NAME, instances)
  while (true) {
    const result = await describeClusters([CLUSTER_NAME])
    if (result.clusters[0].registeredContainerInstancesCount === instances) {
      break
    }
    await sleep(1000)
  }
}
```

---

The addition of `async`/`await` isn't just a bit of syntactic sugar. It's a
language feature that can vastly improve the readability of certain types of
asynchronous code. I'm excited to be able to start using it more.

[node8]: https://nodejs.org/en/blog/release/v8.0.0/
[asyncawait]: https://hackernoon.com/6-reasons-why-javascripts-async-await-blows-promises-away-tutorial-c7ec10518dd9
[loadtesting]: /load-testing
