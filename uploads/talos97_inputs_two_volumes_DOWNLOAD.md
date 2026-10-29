# TALOS97 input NPZ: two volumes

Download both volumes below into one directory. These are binary pieces of one ZIP; merge before extraction.

- [talos97_inputs_97designs_20261029.zip.vol01](https://github.com/kafusukiin2022/hado-mechanical-pilot-artifacts/blob/main/uploads/talos97_inputs_97designs_20261029.zip.vol01) — 92538913 bytes
- [talos97_inputs_97designs_20261029.zip.vol02](https://github.com/kafusukiin2022/hado-mechanical-pilot-artifacts/blob/main/uploads/talos97_inputs_97designs_20261029.zip.vol02) — 92538912 bytes

```bash
cat talos97_inputs_97designs_20261029.zip.vol01 talos97_inputs_97designs_20261029.zip.vol02 > talos97_inputs_97designs_20261029.zip
echo 'ee46eeb3801e6827b2e3499112aba24bdcc0b0a03d7860ea53e6c4ca686ed390  talos97_inputs_97designs_20261029.zip' | sha256sum -c -
unzip talos97_inputs_97designs_20261029.zip -d talos97_inputs
```

Contains all97 input NPZ, including inputs/6hb_009.npz, original68/14/15split,README,per-input SHA256 and provenance manifest. No supervision labels or trajectories. Each volume is below100,000,000bytes. Archive integrity and all97 input hashes verified before splitting; reassemblySHA256 verified.
