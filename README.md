# pthreadpool

[![BSD (2 clause) License](https://img.shields.io/badge/License-BSD%202--Clause%20%22Simplified%22%20License-blue.svg)](https://github.com/Maratyszcza/pthreadpool/blob/master/LICENSE)
[![Build Status](https://img.shields.io/travis/Maratyszcza/pthreadpool.svg)](https://travis-ci.org/Maratyszcza/pthreadpool)

**pthreadpool** is a pthread-based thread pool implementation.
Is is intended to provide functionality of `#pragma omp parallel for` for POSIX systems where OpenMP is not available.

## Features:

* C interface (C++-compatible).
* Run on user-specified or auto-detected number of threads.
* Work-stealing scheduling for efficient work balancing.
* Can be used in Portable Native Client and Native Client environment.
* Extensive unit tests based on **Google Test**.

## Example

  The following example demonstates using the thread pool for parallel addition of two arrays:

```c
static void add_arrays(struct array_addition_context* context, size_t i) {
  context->sum[i] = context->augend[i] + context->addend[i];
}

#define ARRAY_SIZE 4

int main() {
  double augend[ARRAY_SIZE] = { 1.0, 2.0, 4.0, -5.0 };
  double addend[ARRAY_SIZE] = { 0.25, -1.75, 0.0, 0.5 };
  double sum[ARRAY_SIZE];

  pthreadpool_t threadpool = pthreadpool_create(0);
  assert(threadpool != NULL);
  
  const size_t threads_count = pthreadpool_get_threads_count(threadpool);
  printf("Created thread pool with %zu threads\n", threads_count);

  struct array_addition_context context = { augend, addend, sum };
  pthreadpool_compute_1d(threadpool,
    (pthreadpool_function_1d_t) add_arrays,
    (void**) &context,
    ARRAY_SIZE);
  
  pthreadpool_destroy(threadpool);
  threadpool = NULL;

  printf("%8s\t%.2lf\t%.2lf\t%.2lf\t%.2lf\n", "Augend",
    augend[0], augend[1], augend[2], augend[3]);
  printf("%8s\t%.2lf\t%.2lf\t%.2lf\t%.2lf\n", "Addend",
    addend[0], addend[1], addend[2], addend[3]);
  printf("%8s\t%.2lf\t%.2lf\t%.2lf\t%.2lf\n", "Sum",
    sum[0], sum[1], sum[2], sum[3]);

  return 0;
}
```
