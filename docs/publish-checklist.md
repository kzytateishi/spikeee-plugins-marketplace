# Publish checklist

1. Create a new public GitHub repository named `spikeee-plugins-marketplace`.
2. Copy all files from this package into that repository.
3. Validate locally:

```bash
claude plugin validate .
```

4. Test locally from Claude Code:

```text
/plugin marketplace add ./path/to/spikeee-plugins-marketplace
/plugin install figma-to-page@spikeee-plugins
```

5. Push to GitHub.
6. Test from GitHub:

```text
/plugin marketplace add kzytateishi/spikeee-plugins-marketplace
/plugin install figma-to-page@spikeee-plugins
```
