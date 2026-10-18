# Secure Communications — Lab 6: Encrypted image

Recovering the visible content of a bitmap encrypted with AES in ECB mode, without knowing the key. The attack does not touch AES; it relies on ECB mapping identical plaintext blocks to identical ciphertext blocks, so the pattern of repeated blocks in the image survives encryption.

## Files

- `secure-comms-lab-6.ipynb` — the analysis, one step per cell
- `aes.bmp.enc` — the challenge input (encrypted BMP, 921,654 bytes)
- `out.bmp` — the reconstructed image, written by the last cell

## Method

1. **Inspect** — the file starts with `5A 97` instead of `BM` and the header fields decode to nonsense, so the header is encrypted too.
2. **Infer the geometry** — `921,654 − 54 = 921,600 = 640 × 480 × 3`, so a 24-bit 640 × 480 image with 1,920-byte rows.
3. **Find the block grid** — cut the pixel data into 16-byte chunks at each of the 16 possible shifts and count distinct chunks. Shift 10 gives 1,072; the others give 1,511–1,926. The first aligned block starts at file byte 64, so the cipher's grid is anchored at byte 0 of the file.
4. **Count** — with `collections.Counter`, the most common block appears 51,221 times (88.9 %, background) and the second 3,873 times (6.7 %, text); 1,021 blocks appear only once.
5. **Recolour** — map every distinct ciphertext block to a grey value (background white, the rest black, or a frequency-ranked ramp) and replace each block with 16 copies of that value. The 10 head bytes and 6 tail bytes are kept unchanged so the length stays the same.
6. **Rebuild** — write a new 54-byte header for a 640 × 480, 24-bit BMP followed by the recoloured pixel bytes.

## Result

The reconstructed image shows the text `Tro11d`. The original colours are not recovered, only the shape, which is all ECB leaks.

## Running it

```bash
pip install jupyter pillow
jupyter notebook secure-comms-lab-6.ipynb
```

Put `aes.bmp.enc` next to the notebook and run the cells top to bottom.

## Takeaway

The cipher was fine; the mode was the weakness. ECB is deterministic per block and leaks the plaintext's repetition structure. A mode with a fresh IV or nonce (CBC, CTR) hides it, and an authenticated mode such as AES-GCM also protects against tampering.
