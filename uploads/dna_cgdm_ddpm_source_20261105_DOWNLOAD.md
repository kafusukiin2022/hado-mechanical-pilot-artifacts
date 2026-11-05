# DNA-CGDM / DDPM source bundle

Download `dna_cgdm_ddpm_source_20261105.zip` and optionally the standalone Chinese guide `README_ZH.md` and `PACKAGE.json` from this directory.

- ZIP size: 1,058,798 bytes
- SHA-256: `ea4e5259011eec8a51dfd725f6979ed0fbb17be244965c41938c0e1bb622351d`
- Upstream graph-signal-diffusion commit: `018ac033e32105010fe2265e0bdef56db5fa74bb`
- ZIP includes 364 manifest-listed source/config/license/test/evidence files plus `MANIFEST.json`.
- No model weights, checkpoints, MD trajectories, trajectory caches, or environment binaries are included.

Verify and extract:

```bash
sha256sum dna_cgdm_ddpm_source_20261105.zip
unzip -t dna_cgdm_ddpm_source_20261105.zip
unzip dna_cgdm_ddpm_source_20261105.zip
```

The expected ZIP SHA-256 is recorded above and in `PACKAGE.json`. See `README_ZH.md` inside the archive for harness traceability, environment, dependencies, data prerequisites, train/sample commands, verification evidence, and limitations. This is a source/dependency bundle; full training reproduction still requires the project's historical label inputs and MD trajectory data described there.
