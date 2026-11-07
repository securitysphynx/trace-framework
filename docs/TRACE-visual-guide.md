# TRACE Visual Guide

A teaching aid for the [TRACE framework](../README.md).

---

## The Hierarchy of Trust

TRACE dimensions build on each other like a pyramid. Start at the foundation, work up. If any layer cracks, the layers above are suspect.

```
                      ┌───────────────────┐
                      │   E: Explainability   │  Can I explain it?
                      └───────────────────┘
                   ┌─────────────────────────┐
                   │      C: Context         │  Does it fit here?
                   └─────────────────────────┘
              ┌───────────────────────────────────┐
              │        A: Attack Surface          │  Can it be attacked?
              └───────────────────────────────────┘
         ┌─────────────────────────────────────────────┐
         │            R: Reliability                   │  Does it work?
         └─────────────────────────────────────────────┘
    ┌───────────────────────────────────────────────────────┐
    │                  T: Training                          │  What's it built on?
    └───────────────────────────────────────────────────────┘
```

---

## Why This Order?

| Layer | Depends On | Logic |
|-------|------------|-------|
| **T: Training** | — | The foundation. Everything else rests on what the model learned. |
| **R: Reliability** | T | A model can only be as reliable as its training allows. |
| **A: Attack Surface** | T, R | Threat modeling requires understanding how the system works. |
| **C: Context** | T, R, A | Fit can only be assessed once you understand the system itself. |
| **E: Explainability** | All | You can't explain what you don't understand. |

---

## Using This in Training

**Workshop exercise:** Present participants with an AI system scenario. Have them work bottom-to-top:

1. What do we know about **Training**? What don't we know?
2. Given that foundation, what can we say about **Reliability**?
3. Now think adversarially — what's the **Attack Surface**?
4. Does this fit **our** **Context**? Where are the gaps?
5. If something goes wrong, can we **Explain** what happened?

The pyramid makes gaps visible. When someone says "I don't know" at a lower level, it highlights that everything above is uncertain too.

**The exception to teach alongside it:** for closed commercial models, the bottom layer is almost always "I don't know." That does not mean the evaluation stops. When a layer cannot be inspected, it is inferred from the layers above: strong Reliability evidence on your own data tells you something about the Training you cannot see. The pyramid shows where the uncertainty comes from; the substitution rule in the README ("When the Answer Is Unknown") shows how to work with it. Have participants state the inference explicitly: "we cannot see T, so we are relying on R measured here to stand in for it."

---

*Part of the [TRACE Framework](../README.md) by [Melissa Bischoping](https://www.linkedin.com/in/mbischoping/). Licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).*
