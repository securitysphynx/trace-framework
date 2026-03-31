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

---

*Part of the [TRACE Framework](../README.md) by [Melissa Bischoping](https://www.linkedin.com/in/mbischoping/). Licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).*
