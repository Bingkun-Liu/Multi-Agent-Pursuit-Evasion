# Results Summary

This directory collects lightweight final artifacts for the project.

## Figures

```text
figures/F1_bandwidth_sweep_return_capture_step.png
figures/F_intervention_return_heatmap.png
figures/role_occupancy_cross_model_post.png
```

- `F1_bandwidth_sweep_return_capture_step`: compares predator return and first-capture step across communication dimensions.
- `F_intervention_return_heatmap`: shows how communication cutoff/intervention affects return across bandwidths.
- `role_occupancy_cross_model_post`: visualizes role occupancy patterns across no-communication, communication, and zero-message settings.

PDF versions are included for report-quality rendering.

## Tables

```text
tables/bandwidth_sweep_table.tex
tables/cutoff_intervention_table.tex
```

These LaTeX tables summarize the numerical results used in the report.

## Notes

The full raw outputs, model checkpoints, and intermediate analysis files remain under:

```text
onpolicy/scripts/results/MPE/simple_tag/mappo/
```

For a lightweight final repository, keep this summary directory in Git and move large checkpoints or raw logs outside the repository.
