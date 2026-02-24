# NoSQL Injection Lab -- Full Logical Breakdown + Burp Guide

------------------------------------------------------------------------

# 📌 Overview

This document explains in depth:

1.  What happened internally in the lab
2.  How operator precedence changed the logic
3.  Why the expression became always true
4.  How to interpret the final HTML response
5.  How Burp Suite was used

------------------------------------------------------------------------

# 🧠 1. Original Backend Logic

The backend likely evaluated something similar to:

    return this.category == 'Gifts' && this.released == true

Meaning:

-   Product must belong to category "Gifts"
-   AND product must be released
-   Both must be true

------------------------------------------------------------------------

# 🔎 2. Boolean Logic Refresher

## AND (&&)

    true && true   → true
    true && false  → false
    false && true  → false
    false && false → false

If any part is false → entire expression is false.

------------------------------------------------------------------------

## OR (\|\|)

    true || false  → true
    false || true  → true
    false || false → false

If any part is true → entire expression is true.

------------------------------------------------------------------------

# 🔥 3. What the Injection Actually Produced

We injected:

    Gifts'||1||'

The server then evaluated something equivalent to:

    this.category == 'Gifts'||1||'' && this.released == true

------------------------------------------------------------------------

# 🧠 4. Operator Precedence (CRITICAL)

In JavaScript:

-   && has higher precedence than \|\|

-   So:

    A \|\| B && C

Is interpreted as:

    A || (B && C)

NOT:

    (A || B) && C

------------------------------------------------------------------------

# 🔬 5. Step‑by‑Step Evaluation

Expression:

    this.category == 'Gifts' || 1 || '' && this.released == true

Apply precedence:

    this.category == 'Gifts' || 1 || ('' && this.released == true)

Now evaluate:

1)  '' is falsy
2)  false && anything → false

So it becomes:

    this.category == 'Gifts' || 1 || false

Now evaluate OR:

1 is truthy.

    anything || 1 || anything → true

Final result:

    true

The entire condition becomes true for ALL products.

------------------------------------------------------------------------

# 💥 6. Why released == true Became Irrelevant

We did NOT remove it.

We made the entire expression evaluate to true before it mattered.

If the condition returns true, the product passes the filter regardless
of release status.

------------------------------------------------------------------------

# 📄 7. Interpreting the Final HTML Response

Before injection:

    <section class="container-list-tiles">
        (3 products)
    </section>

After injection:

    <section class="container-list-tiles">
        (20 products across multiple categories)
    </section>

This proves:

-   Category restriction bypassed
-   Released restriction bypassed
-   Entire catalog returned

The increase in
```{=html}
<div>
```
elements inside the product container confirms filter bypass.

------------------------------------------------------------------------

# 🛠️ 8. How Burp Suite Was Used

## Proxy

Captured:

    /filter?category=Gifts

## Repeater

-   Modified category parameter
-   Tested boolean conditions
-   Compared response differences

## Logical Tests

False test:

    Gifts' && 0 && 'x

Returned zero products.

True test:

    Gifts' && 1 && 'x

Returned products.

Final bypass:

    Gifts'||1||'

Returned full catalog.

------------------------------------------------------------------------

# 🚨 9. Core Lessons

1.  Backend errors reveal execution context.
2.  Boolean testing confirms logic execution.
3.  Operator precedence can collapse security checks.
4.  OR true bypasses AND restrictions.
5.  HTML comparison confirms backend logic changes.

------------------------------------------------------------------------

# 🛡️ 10. Defensive Lessons

-   Never use \$where with user input.
-   Avoid dynamic JavaScript evaluation in MongoDB.
-   Enforce strict input validation.
-   Use structured queries.
-   Sanitize input.

------------------------------------------------------------------------

# 🎯 Final Mental Model

Original:

    A && B

After injection:

    A || 1 || ('' && B)

Simplifies to:

    true

Once you understand this transformation, you understand the
vulnerability.

------------------------------------------------------------------------

End of writeup.

