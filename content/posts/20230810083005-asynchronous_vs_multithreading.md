---
title: "Asynchronous vs Multithreading"
draft: false
---

## Asynchronous vs Multithreading {#asynchronous-vs-multithreading}

From the definitions we just provided, we can see that [multithreading programming]({{< relref "20230810082726-multithreading_programming.md" >}}) is all about [concurrent]({{< relref "20230810074202-concurrent_programming.md" >}}) execution of different functions. [Asynchronous programming]({{< relref "20230810082215-asynchronous_programming.md" >}}) is about non-blocking execution between functions, and we can apply async with single-threaded or multithreaded programming.

So, multithreading is one form of asynchronous programming.

Let’s take a simple analogy; you have a friend, and you decided to make dinner together.

Async is when you say to your friend, “You go to the store and buy pasta. Let me know when you get back, to make dinner together. Meanwhile, I’ll prepare sauce and drinks.”

Threading is, “You boil the water. I’ll heat the tomato sauce. While the water is boiling, ask me and I’ll put the pasta in. When the sauce is hot, you can add cheese. When both are done, I’ll sit down and you serve dinner. Then we eat.”. In the threading analogy, we can see the sequence of “When, Do” events, which represent the sequential set of instructions per each person (thread).

From that analogy, we can conclude that Multithreading is about workers, Asynchronous is about tasks.


## Reference List {#reference-list}

1.  <https://www.baeldung.com/cs/async-vs-multi-threading#asynchronous-vs-multithreading>
