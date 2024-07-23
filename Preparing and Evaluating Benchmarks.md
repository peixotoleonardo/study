 **Anything can affect your benchmark**, the best you can do is reduce the variability as much as you can. Programs are, eventually, non-deterministic and their execution time varies based on several system variables and conditions. Such variation makes it uncertain what the effect of a particular optimization might be.

> [!tip] Reducing variability is essential to get reliable results.

Therefore, it is strongly recommended to:

- Use a **dedicated server** to perform benchmarks
- Reduce any turbulence that might affect your benchmark result — One can use htop to see background processes and then, close them
- Try to run the benchmark as closely as possible to the production environment

**Isolate your Microbenchmarks**

A common **mistake** by developers is to **not reset the environment between each benchmark run**. Node.js for instance uses V8 as its JavaScript engine and as a runtime that contains a JIT (Just-in-Time) compiler, the execution order has a direct impact on perf optimizations.

Look at the following snippet:

```js
const operations = 1_000_000

const test = (func) => {
  const start = performance.now()

  for(let i = 0; i < operations; i++) {
    func()
  }

  return `${(performance.now() - start).toFixed(7)}ms`
}

console.log(`Operations: ${operations}`)

console.log('arrow function', test(() => {
  const a = () => {}
  a()
}))

console.log('regular function', test(() => {
  const b = function () {}
  b()
}))
```

## Evaluating Results

Usually, when realizing performance tweaks on an application, a common workflow is:

1. Run a benchmark before the change
2. Run a benchmark after the change
3. Compare the first run against the second run

Even reducing variability, some benchmarks simply vary. Therefore, you may ask:

- “How many times should I run a benchmark?”

The answer depends on the variance interval. The [[Rigorous Benchmarking in Reasonable Time]] is an excellent resource on this topic, this paper shows how to establish the repetition count necessary for any evaluation to be reliable.

[[Student’s test]] is a statistical method used in the testing of the null hypothesis(H0) for the comparison of means between groups. Running a _t-test_ helps you to understand whether the differences are [[Statistically Significant| statistically significant]] — However, if performance improvements are large, 2x more, for example, there is no need for statistical machinery to prove they are real.

While computing a confidence interval, the number of samples _n_ (benchmark executions) are categorized into two groups:

1. **n** is large (usually ≥ 30).
2. **n** is small (usually < 30).

This article approaches the first group (_n ≥ 30_) — Both groups are covered in detail in the paper Statistically [[Rigorous Java Performance Evaluation]] - section 3.2. The module [`ttest`](https://www.npmjs.com/package/ttest) will abstract the confidence calculation. In case you are interested in the equation.

> [!warning]
> The Student’s t-test approach relies on the mean of each group. When dealing with HTTP Benchmarks, outliers can happen, making the mean useless info, so be careful with the mean. Always plot your data into a graph so you can understand its behaviour.

```js
```js
const ttest = require('ttest')

const A = [
  46.37, 45.43, 45.1, 43.25, 45.51,
  46.8,  45.3, 43.58, 43.3, 45.42,
  46.02, 44.5, 43.94, 43.67, 43.55,
  43.71, 46.62, 46.56, 43.5, 43.84,
  45.75, 43.86, 46.76, 43.32, 44.08,
  45.92, 46.2, 46.24, 43.97, 43.03
]

const B = [
  45.69,  43.3, 45.16, 44.66,
  42.27, 42.83, 43.28, 43.01,
  43.37, 44.64, 44.85, 44.61,
  42.05, 44.01,  43.9, 42.39,
  42.24, 45.22, 45.66, 45.31,
  45.33, 45.02, 43.26, 44.43,
  45.53, 42.19, 42.44, 43.66,
  44.66, 45.55
]


const res = ttest(A, B, { varEqual: true, alpha: 0.05 })

if (res.pValue() <= 0.05) {
  console.log(`It's a significant difference`)
} else {
  console.log(`It's NOT a significant difference`)
}

console.log('Confidence', res.confidence())
```

This analysis enables one to determine whether differences observed in measurements are due to random fluctuations in the measurements or due to actual differences in the alternatives compared against each other. Typically, 5% is a threshold used to identify actual differences.

As a probabilistic test, the current result allows you to say: “I am 95% sure my optimization makes a difference”.

Do not forget, the benchmark insights come from the difference between branches instead of raw values and even using a probabilistic test is extremely important to know your data, plotting them into a graph is always helpful.

**Be realistic in your benchmarks**

Sometimes the benchmark result is totally accurate, but, the way they are shared is tendentious.

Normally any performance improvement is welcome, but in some circumstances, the complexity or disadvantages of implementing the _faster_ approach may not be worth it.

**Benchmark results can tell you more than performance gotchas**

Through the results and the active benchmark, it is possible to predict the software limitations.

Let’s say you are looking at an existing system currently performing a thousand requests per second. The busiest resources are the 2 CPUs, which are averaging 60% utilization; Therefore, through basic math you can find a potential limitation using the following equation:

```
n = CPU count (2)
CPU% per request = n * total CPU%/requests (0.12% CPU per request)
Max request/s = 100% x n/CPU% per request (1665 req/sec)
```

CPU% per request = 2 x 60%/1000 = 0.12% CPU per request

Max requests/s = 200/0.12 = 1665 req/sec approximately.

This is a common supposition for CPU Bound applications, however, it ignores the fact that other resources can reach their limitation before the CPU. Therefore, 1665 req/sec can be considered the maximum req/sec this application can achieve before reaching CPU Saturation.

## References

- https://blog.rafaelgss.dev/preparing-and-evaluating-benchmarks