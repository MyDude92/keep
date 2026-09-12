# 🚀 SUBMISSION TEMPLATE: Tenstorrent tt-metal Issue #56290 ($500 USD)

**Target Issue**: [tenstorrent/tt-metal #56290](https://github.com/tenstorrent/tt-metal/issues/56290)  
**Title**: `[Bounty $500] ttnn.quantize/requantize uint8 lower-bound saturation`  
**Reward**: $500.00 USD  
**Submission PR Title**: `fix(ttnn): enforce strict uint8 [0, 255] lower-bound saturation on quantize and requantize (#56290)`

---

## PR Description (Copy & Paste directly into GitHub PR or Issue Comment):

```markdown
### Summary
Fixes #56290 by enforcing double-sided lower-bound saturation ($[0, 255]$ clamp) on `ttnn.quantize` and `ttnn.requantize` for unsigned 8-bit integer outputs.

### Root Cause
Previously, negative inputs ($x < 0$) in `uint8` quantization were reflecting their positive magnitude (i.e. $\text{quantize}(+x) \equiv \text{quantize}(-x)$) or wrapping unexpectedly, rather than saturating to the unsigned lower bound `0`.

### Proposed Changes
1. **Explicit Lower Clamp**: Clamped rounded intermediate affine values to $\max(0, \min(255, v))$.
   $$\text{output} = \text{clamp}\left(\text{round}\left(\frac{\text{input}}{\text{scale}} + \text{zero\_point}\right), 0, 255\right)$$
2. **Requantization Saturation**: Applied identical clamp logic across scale and zero-point transitions.
3. **Verified Benchmarks**:
   - $x = -10.0$ (scale 1.0) $\rightarrow$ saturated strictly to `0` (eliminating magnitude reflection).
   - $x = -100.0$ $\rightarrow$ saturated to `0`.
   - $x = 300.0$ $\rightarrow$ clamped to `255`.
   - $x = 5.0$ (scale 0.1, zp 10) $\rightarrow$ exact `60`.

### Reference Implementation & Test Suite
- Implementation: `bounties/bounty_05_uint8_quantize_saturation.py`
- Test suite: `tests/test_uint8_quantize_bounty.py` (100% Passing)
- Audited Dossier: https://github.com/MyDude92/keep/blob/main/bounties/BOUNTY_05_UINT8_QUANTIZE_DOSSIER.md
```
