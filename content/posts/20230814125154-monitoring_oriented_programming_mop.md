---
title: "Monitoring-Oriented Programming(MOP)"
draft: false
---

## Monitoring-Oriented Programming {#monitoring-oriented-programming}

Monitoring-Oriented Programming, abbreviated MOP, is a software development and analysis framework aiming at reducing the gap between formal specification and implementation by allowing them together to form a system.
In MOP, runtime monitoring is supported and encouraged as a fundamental principle for building reliable software: monitors are automatically synthesized from specified properties and integrated into the original system to check its dynamic behaviors during execution. When a specification is violated or validated at runtime, user-defined actions will be triggered, which can be any code from information logging to runtime recovery. One can understand MOP from at least three perspectives: as a discipline allowing one to improve safety, reliability and dependability of a system by monitoring its requirements against its implementation at runtime; as an extension of programming languages with logics (one can add logical statements anywhere in the program, referring to past or future states); and as a lightweight formal method.


## The MOP Matrix {#the-mop-matrix}

MOP is generic both in the underlying programming language and in the requirements specification formalism in which properties are expressed. The clickable table below shows all current instances of MOP and can be used to navigate MOP's webpages on this site. The rows show instances by the language and the columns instances by the logic. A cell "..." means that the corresponding combination is not yet available.

|                                                       | FSM     | ERE          | CFG     | PTLTL              | LTL     | PTCaRet     | SRS     | ... |     |
|-------------------------------------------------------|---------|--------------|---------|--------------------|---------|-------------|---------|-----|-----|
| [JavaMOP]({{< relref "20230815122557-javamop.md" >}}) | JavaFSM | JavaERE      | JavaCFG | JavaPTLTL          | JavaLTL | JavaPTCaRet | JavaSRS | ... |     |
| BusMOP                                                | BusFSM  | BusERE       | ...     | BusPTLTL           | ...     | ...         | ...     | ... |     |
| ROSMOP                                                |         | N/A (ROSFSM) | ...     | N/A (ROSCFGPlugin) | ...     | ...         | ...     | ... | ... |


## MOP Language Instances {#mop-language-instances}

[JavaMOP]({{< relref "20230815122557-javamop.md" >}}) is an instance of MOP for Java.


## Reference List {#reference-list}

1.  <https://javamop.coecis.cornell.edu/resources#:~:text=Monitoring-Oriented>
