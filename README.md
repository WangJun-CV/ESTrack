# ESTrack
## About This Project

This repository provides the official implementation of our paper:

> **"[ESTrack: Enhanced Visual Tracking via Spatial-Channel Summation Attention Network]"**  
> *Authors: Yuanyun Wang, Shenmiao Jin, Zhuo An, Jun Wang*  
> *Submitted to: The Visual Computer*  
> *Status: Under Review*

If you use this codebase or find our work helpful in your research, please kindly cite our manuscript:
```bibtex
@article{Wang2025ESTrack,
  title={ESTrack: Enhanced Visual Tracking via Spatial-Channel Summation Attention Network},
  author={Yuanyun Wang, Shenmiao Jin, Zhuo An, Jun Wang},
  journal={Submitted to The Visual Computer},
  year={2025},
  note={Under Review},
  url={https://}
  }
```

## Prepare Environment

Conda env: Use the Anaconda (CUDA 11.0)
```
conda create -n ESTrack python=3.8
conda activate ESTrack
bash install.sh
```

## Set project paths
Run the following command to set paths for this project
```
python tracking/create_default_local_file.py --workspace_dir . --data_dir ./data --save_dir ./output
```
After running this command, you can also modify paths by editing these two files
```
lib/train/admin/local.py  # paths about training
lib/test/evaluation/local.py  # paths about testing
```

## Dataset Preparation
Put the tracking datasets in ./data. It should look like this:
```
${PROJECT_ROOT}
 -- data
     -- lasot
         |-- airplane
         |-- basketball
         |-- bear
         ...
     -- got10k
         |-- test
         |-- train
         |-- val
     -- coco
         |-- annotations
         |-- images
     -- trackingnet
         |-- TRAIN_0
         |-- TRAIN_1
         ...
         |-- TRAIN_11
         |-- TEST
``` 
## Training
Download pre-trained [CAE ViT-Base weights](https://github.com/lxtGH/CAE)(cae_base.pth) and put it under  `$PROJECT_ROOT$/pretrained_models`.   

Run the command below to train the model:
```sh
python lib/train/run_training.py --script ESTrack --config cae_center_all_ep300 --save_dir ./output --mode multiple --nproc_per_node 4  --use_wandb 1
```
- If you want to use wandb to record detailed log and get clear curve of training, set `--use_wandb 1`.  
- The batch size, learning rate and the other detailed parameters can be set in config file, e.g., `experiments/ESTrack/cae_center_all_ep300.yaml`.
- You can also replace `--config` with the desired model config under `experiments/ESTrack`.

## Evaluation
Use your own training weights or ours([Baidu Drive](https://pan.baidu.com/s/1EpOHDlhp8ORUG5fWJwYVNQ?pwd=snbu)) in `$PROJECT_ROOT$/output/checkpoints/train/ESTrack`.  

Change the corresponding values of `lib/test/evaluation/local.py` to the actual benchmark saving paths

Some testing examples:

- LaSOT or other off-line evaluated benchmarks (modify `--dataset` correspondingly)
```sh
python tracking/test.py ESTrack cae_center_all_ep300 --dataset lasot --threads 0 --num_gpus 4 --ep 300
python tracking/analysis_results.py # need to modify tracker configs and names
```

- GOT10K-test
```sh
python tracking/test.py ESTrack cae_center_all_ep300 --dataset got10k_test --threads 0 --num_gpus 4 --ep 300
python lib/test/utils/transform_got10k.py --tracker_name ESTrack --cfg_name cae_center_all_ep300_300 # the last number is epoch
```

- TrackingNet
```sh
python tracking/test.py ESTrack cae_center_all_ep300 --dataset TrackingNet --threads 0 --num_gpus 4 --ep 300
python lib/test/utils/transform_trackingnet.py --tracker_name ESTrack --cfg_name cae_center_all_ep300_300 # the last number is epoch
```

Our tracking raw results are there([Baidu Drive](https://pan.baidu.com/s/11eH2V3c2F6q9PtDpjfZS3g?pwd=c4fh)).
## Test FLOPs, and Speed
*Note:* The speeds reported in our paper were tested on a single RTX3060 GPU.

```
python tracking/profile_model.py --script DSATrack --config cae_center_all_ep300
```

## Acknowledgement
Thanks for the [OSTrack](https://github.com/botaoye/OSTrack) and [PyTracking](https://github.com/visionml/pytracking) library, which helps us to quickly implement our ideas.