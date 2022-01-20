*View this file with results and syntax highlighting [here](https://mlochbaum.github.io/BQN/help/insert.html).*

# Double Acute Accent (`˝`)
    
## `𝔽˝ 𝕩`: Insert
    
Fold over `𝕩` with `𝔽` from right to left i.e. Insert `𝔽` between the major cells of `𝕩`.
    
           a ← 3‿3 ⥊ ↕9

           +˝ a

           0‿1‿2 + 3‿4‿5 + 6‿7‿8

    
## `𝕨 𝔽˝ 𝕩`: Insert With initial
    
Monadic insert, but use `𝕨` as initial right argument.

If 
    
           a ← 3‿3 ⥊ ↕9

           1‿1‿1 +˝ a

           1 +˝ a

           0‿1‿2 + 3‿4‿5 + 6‿7‿8 + 1‿1‿1

           

    