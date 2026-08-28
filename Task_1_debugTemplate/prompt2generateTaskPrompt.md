Convert this request into a task prompt that I can give to a ai-agent system to execute:
1.
For code runner agent to do:
- run command2, comamnd2p, and command3 from stage1 with this setup:
```
stage1_label="Run2_NanoV15_forVBFChannel_July06_2026_jetUncRedo"
years="2018,2017,2016postVFP,2016preVFP"
time(bash stage1_loop_Improved.sh -c configs/datasets/dataset_nanoAODv15_run2.yaml -v 15 -l  $stage1_label -y $years -m 2 -k)
time(bash stage1_loop_Improved.sh -c configs/datasets/dataset_nanoAODv15_run2.yaml -v 15 -l  $stage1_label -y $years -m 2p -k)
time(bash stage1_loop_Improved.sh -c configs/datasets/dataset_nanoAODv15_run2.yaml -v 15 -l  $stage1_label -y $years -m 3 -k)
```
and with "combine" pixi env, run:
```
sh produce_combine_cards.sh {stage3_output_path} 2018
sh produce_combine_cards.sh {stage3_output_path} 2017
sh produce_combine_cards.sh {stage3_output_path} 2016postVFP
sh produce_combine_cards.sh {stage3_output_path} 2016preVFP
sh produce_combine_cards.sh {stage3_output_path} Run2

bash produce_significance.sh {stage3_output_path} 2018 
bash produce_significance.sh {stage3_output_path} 2017 
bash produce_significance.sh {stage3_output_path} 2016postVFP 
bash produce_significance.sh {stage3_output_path} 2016preVFP 
bash produce_significance.sh {stage3_output_path} Run2 
```
- note down the output paths for later review by validation agent.

2.
for reviewer agent to do:
- find the outputs of coder runner agent
- mc sample names among year groups (Run2 and Run3) are same
- the comparison yields for each mc sample (ie from datacard after stage3) among data-taking years roughly match the integrate lumi ratio among data-taking years
- make sure that the year by year significances  values (saved in `significance.txt`) scales as expected from integrated lumi values unless there's an explicit physics/detector reason why.
- the up and down histograms should have similar delta compared to other systematic variations (unless specified). if one variation is very different from nominal histogram compared to other variations, raise a red flag
- run the up and down python plot (for human inspection)

3. When reviewer agent reviews, it is likely to find a bug, then it gives a report.

4. The code generator reads the report from the reviewer agent, makes the necessary changes
5.  repeat process until converged

6. once converged, save git diff for overall summary inspection