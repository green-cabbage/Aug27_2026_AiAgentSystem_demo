write me the task prompt for the ai-agent system for implementing a new feature.
when you write/generate the task, include the --use_gateway command in allowed_commands, and flip execution_granted (and production_granted since this is a full-scale run, not a toy test) to true.


---

## code generator task:
- Implement a new feature: a scale factor correction for MET phi variable. The Data/MC histogram of MET phi variable is not flat, and thus needs to be improved.
- The code implementation for scale factors are done using (from coffea.analysis_tools.Weights) over MC samples at `src/copperhead_processor.py` 
- I have already started implementing the scale factor code. You can find it in `new_features/MET_phi_modulation/starting_code/met_phi_correction_mwe.py`. But I am missing:
    - Jet_jecL1
    - Jet_jecL1L2L3
    - CorrT1METJet_jecL1
    - CorrT1METJet_jecL1L2L3
    - ways to calculate uncertainty of the SF
    - etc
- Use ref materials saved in `new_features/MET_phi_modulation/ref_material/` and materials in this directory to find the correct solution. For your ref, the pseudo code for this implmentation exists in slides 5,6 of `new_features/MET_phi_modulation/ref_material/260202_JMEGeneral_Type1METWithNano_Nurfikri.pdf`


## code runner task:
- once the code is generated, restart the dask cluster. To restart dask cluster, prompt
```
close existing dask cluster and start a new dask cluster with parameters defined in dask_cluster_params.txt and scale to 100 workers
```
- once the cluster is restarted, run stage1 for 2016preVFP, dy (skip dy_VBF_filter) and data only with script below over pixi env. Make note of the stage1_output for the next step
```
bash stage1_loop_Improved.sh -c configs/datasets/dataset_nanoAODv15_run2.yaml -v 15 -l  {stage1_label} -y "2016postVFP" -m 1 -k
bash stage1_loop_Improved.sh -c configs/datasets/dataset_nanoAODv15_run2.yaml -v 15 -l  {stage1_label} -y "2016postVFP" -m compact -k

```
- once complete, edit the relevant parameters in `run_plotter.py` (ie stage1_output path) and run script below over pixi env
```
python run_plotter.py
```
- make note of the pdf and txt save path

## reviewer task:
- review the contents of txt outputs from the previous step.
- the stage1_output label to compare to is `Run2_NanoV15_forVBFChannel_July06_2026_jetUncRedo`
- check that the flatness of data/MC histograms is improved over z-peak region for MET phi over nocat, ggh and vbf categories, but mainly over nocat category.
- check that there's no significantly worsening or generating a shape artifact on data/MC for other variables 
- if checks fail, start the loop again by notifying the code generator
- note down the plot save path in your reports so that humans can inspect the pdf plots later


