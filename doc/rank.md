*View this file with results and syntax highlighting [here](https://mlochbaum.github.io/BQN/doc/rank.html).*

# Cells and Rank

The Cells modifier `˘` applies a function to major cells of its argument, much like [Each](map.md) applies to elements. Each result from `𝔽` becomes a major cell of the result, which means they must all have the same shape.

The Rank modifier `⎉` generalizes this concept by allowing numbers provided by `𝔾` to specify a rank for each argument: non-negative to indicate the rank of each array passed to `𝔽`, or negative for the number of axes that are mapped over. Cells, which maps over one axis of each argument, is identical to `⎉¯1`. Rank is analogous to the [Depth modifier](depth.md#the-depth-modifier), but the homogeneous structure of an array eliminates some tricky edge cases found in Depth.

## Cells

The function Cells (`˘`) is named after *major cells* in an array. A major cell is a component of an array with dimension one smaller, so that the major cells of a list are [units](enclose.md#whats-a-unit), the major cells of a rank-2 table are its rows (which are lists), and the major cells of a rank-3 array are tables.

The function `𝔽˘` applies `𝔽` to the major cells of `𝕩`. So, for example, where [Nudge](shift.md) (`»`) shifts an entire table, Nudge Cells shifts its major cells, or rows.

        a ← 'a' + 3‿∘ ⥊ ↕24  # A character array

        ⟨  a      ,     »a     ,    »˘a ⟩

What's it mean for Nudge to shift the "entire table"? The block above shows that it shifts downward, but what's really happening is that Nudge treats `𝕩` as a collection of major cells—its rows—and shifts these. So it adds an entire row and moves the rest of the rows downwards. Nudge Cells appears similar, but it's acting independently on each row, and the values that it moves around are major cells of the row, that is, rank-0 units.

Here's an example showing how Cells can be used to shift each row independently, even though it's not possible to shift columns like this (in fact the best way to do that would be to [transpose](transpose.md) in order to work on rows). It uses the not-yet-introduced dyadic form of Cells, so you might want to come back to it after reading the next section.

        (↑"∘∘") ⊑⊸»˘ a

You can also see how Cells splits its argument into rows using a less array-oriented primitive: [Enclose](enclose.md) just wraps each row up so that it appears as a separate element in the final result.

        <˘ a

Enclose also comes in handy for the following task: join the rows in an array of lists, resulting in an array where each element is a joined row. The obvious guess would be "join cells", `∾˘`, but it doesn't work, because each `∾` can return a result with a different length. Cells tries to make each result of `∾` into a *cell*, when the problem was to use it as an *element*. But a 0-cell is an enclosed element, so we can close the gap by applying `<` to a joined list: `<∘∾`.

        ⊢ s ← "words"‿"go"‿"here" ≍ "some"‿"other"‿"words"

        ∾˘ s

        <∘∾˘ s

This approach can apply to more complicated functions as well. And because the result of `<` always has the same shape, `⟨⟩`, the function `<∘𝔽˘` can never have a shape agreement error. So if `𝔽˘` fails, it can't hurt to check `<∘𝔽˘` and see what results `𝔽` is returning.

### Two arguments

When given two arguments, Cells tries to pair their cells together. Starting simple, a unit (whether array or atom) on either side will be paired with every cell of the other argument.

        '∘' »˘ a

If you *want* to use this one-to-many behavior with an array, it'll take more work: since you're really only mapping over one argument, [bind](hook.md) the other inside Cells.

        "∘∘" »˘ a

        "∘∘"⊸»˘ a

This is because the general case of Cells does one-to-one matching, pairing the first axis of one argument with the other. For this to work, the two arguments need to have the same length.

        ⟨ "012" »˘ a,  (3‿∘⥊"UVWXYZ") »˘ a ⟩

The arguments might have different ranks: for example, `"012"` has rank 1 and `a` has rank 2 above. That's fine: it just means Cells will pass arguments of rank 0 and 1 to its operand. You can see these arguments using [Pair](pair.md) Cells, `⋈˘`, so that each cell of the result is just a list of the two arguments used for that call.

        "012" ⋈˘ a

## Rank

Rank (`⎉`) is a generalization of Cells (`𝔽˘` is defined to be `𝔽⎉¯1`) that can apply to arbitrary—not just major—cells and combine different levels of mapping for two arguments.

Rank comes in handy when there are high-rank arrays with lots of exciting axes, which is a great use case for BQN but honestly isn't all that common. And to continue this trend of honesty, using Rank just never *feels* good—it's some heavy machinery that I drag out when nothing else works, only to make use of a small part of the functionality. If Cells covers your use cases, that's probably for the best!

### Negative and positive ranks

I've said that `𝔽⎉¯1` is `𝔽˘`. And it's also the case that `𝔽⎉¯2` is `𝔽˘˘`. And `𝔽⎉¯3` is `𝔽˘˘˘`. And so on.

        (↕4) (⋈˘˘˘ ≡ ⋈⎉¯3) ↕4‿2‿2‿5

So `𝔽⎉(-k)`, at least for `k≥1`, is how you map over the first `k` axes of an array or two. We'll get more into why this is in the next section. What about some positivity for a change?

        <⎉0 "abc"≍"def"

        <⎉1 "abc"≍"def"

The function `𝔽⎉k`, for `k≥0`, operates on the `k`-cells of its arguments—that is, it maps over all *but* the last `k` axes. For any given argument `a`, ranks `k` and `k-=a` are the same, as long as `k≥0` and `(k-=a)≤¯1`. So rank 2 is rank ¯3 for a rank-5 array. The reason this option is useful is that the same rank might be applied to multiple arguments, either with multiple function calls or one call on two arguments. Let's revisit an example with Cells from before, shifting the same string into each row of a table. The function `»` should be called on rank-1 strings, but because the argument ranks are different, a negative rank can't get down to rank 1 on both sides. Positive rank 1 does the job, allowing us to unbundle the string `"∘∘"` so that `»⎉1` is a standalone function.

        "∘∘"⊸»˘ a

        "∘∘" »⎉1 a

The rank for a given argument is clamped, so that on a rank 3 argument for example, a rank of ¯5 counts as ¯3 and a rank of 6 counts as 3 (same for any other value less than ¯3 or greater than 3, although it does have to be a whole number). You may have noticed there's [no](../commentary/problems.md#rankdepth-negative-zero) option for ¯0, "don't map over anything", but ∞ serves that purpose as it indicates the highest possible rank and thus the entire array. More on why that's useful later.

### Frame and Cells

### Multiple and computed ranks

The Rank modifier also accepts a list of one to three numbers for `𝕘`, as well as a function `𝔾` returning such a list. Practically speaking, here's what you need to know:

- A single number or one-element list indicates the ranks for all arguments.
- Two numbers indicate the ranks for `𝕨` and `𝕩`.

        ⊢ m ← >⟨0‿1‿0,¯1‿0‿0,0‿0‿1⟩

        m +˝∘×⎉1‿∞ 1‿2‿3

        m +˝∘×⎉1‿∞ 1‿2‿3×⌜1‿10

        ("abc"≍"def") ∾⎉1⎉1‿∞ >"QR"‿"ST"‿"UV"

Here's the full, boring description of how `𝔾` is handled. The operand `𝔾` is called on the arguments `𝕨𝔾𝕩` before doing anything else (if it's not a function, this just returns `𝕘`). Then it's converted to a list. It's required to have rank 0 or 1, but numbers and enclosed numbers are fine. This list can have one to three elements; three elements is the general case, as the elements give the ranks for monadic `𝕩`, dyadic `𝕨`, and dyadic `𝕩` in order. If there are less than three elements, the list `r` is expanded backwards-cyclically to `3⊸⥊⌾⌽r`, turning `⟨a⟩` into `a‿a‿a` and `a‿b` into `b‿a‿b`. So `3⊸⥊⌾⌽⥊𝕨𝔾𝕩` is the final formula.

### Leading axis agreement