# Youtube-8M-WILLOW
This is the solution of the kaggle Youtube-8M Large-Scale Video Understanding challenge winners (https://www.kaggle.com/c/youtube8m).
For more details about our models,  please read our arXiv paper: ... .

The code is built on top of the Google Youtube-8M starter code (https://github.com/google/youtube-8m)
Please look at their README to see the needed dependencies to run the code.

You will additionally only need to have the pandas python library installed.

Hardware requirement: Each model can be run on a single NVIDIA TITAN X 12 GB GPU. Some of the trained
do not fit with a GPU with at least 10GB of memory. In that case, please do not modify the training batch size 
of these models as it might affect the final results.

Our best submitted model (GAP: 0.84967% on the private leaderboard) is a weighted ensemble of 25 models.
However for the sake of simplicity, this repot will contain a much more simple ensemble of 
7 models that is enough to reach the first place with a significant margin (GAP ~ 84.7%)

Please note that because of the time constraint, we did not have time to try to run the code from scratch.
It might be possible, but rather unlikely, that something is not working properly. If so please contact me.

## Training the single models

Each of the following command lines train a single model. They are scheduled to stop training at the good time.

Our model were trained on all the training set and almost all the validation set (we only discarded 21k videos). We will however, train the models on all both training and validation set as it was allowed in the kaggle competition. It should not make any huge difference.

Each model takes several days to train, so each command line are separated in order to be run in parallel if possible. 
Please replace 'path_to_features' with the folder path which contains all the tensorflow record frame level feature.
```sh
path_to_features='path_to_features'
```

Training Gated NetVLAD (256 Clusters):

```sh
python train.py --train_data_pattern='$path_to_features/*a*??.tfrecord' --model=NetVLADModelLF --train_dir=gatednetvladLF-256k-1024-80-0002-300iter-norelu-basic-gatedmoe --frame_features=True --feature_names="rgb,audio" --feature_sizes="1024,128" --batch_size=80 --base_learning_rate=0.0002 --netvlad_cluster_size=256 --netvlad_hidden_size=1024 --moe_l2=1e-6 --iterations=300 --learning_rate_decay=0.8 --netvlad_relu=False --gating=True --moe_prob_gating=True --max_step=700000
```

Note: The best single model is this model but with flag --max_step=300000. We somehow need it to train longer for better effect on the ensemble.

Training Gated NetFV (128 Clusters):


```sh
python train.py --train_data_pattern='$path_to_features/*a*??.tfrecord' --model=NetFVModelLF --train_dir=gatednetfvLF-128k-1024-80-0002-300iter-norelu-basic-gatedmoe --frame_features=True --feature_names="rgb,audio" --feature_sizes="1024,128" --batch_size=80 --base_learning_rate=0.0002 --fv_cluster_size=128 --fv_hidden_size=1024 --moe_l2=1e-6 --iterations=300 --learning_rate_decay=0.8 --fv_relu=False --gating=True --moe_prob_gating=True --fv_couple_weights=False --max_step=600000
```

Training Gated Soft-Dbof (4096 Clusters):

```sh
python train.py --train_data_pattern='$path_to_features/*a*??.tfrecord' --model=GatedDbofModelLF --train_dir=gateddboflf-4096-1024-80-0002-300iter --frame_features=True --feature_names="rgb,audio" --feature_sizes="1024,128" --batch_size=80 --base_learning_rate=0.0002 --dbof_cluster_size=4096 --dbof_hidden_size=1024 --moe_l2=1e-6 --iterations=300 --dbof_relu=False --max_step=1000000
```

Training Soft-Dbof (8000 Clusters):

```sh
python train.py --train_data_pattern='$path_to_features/*a*??.tfrecord' --model=SoftDbofModelLF --train_dir=softdboflf-8000-1024-80-0002-300iter --frame_features=True --feature_names="rgb,audio" --feature_sizes="1024,128" --batch_size=80 --base_learning_rate=0.0002 --dbof_cluster_size=8000 --dbof_hidden_size=1024 --iterations=300 --dbof_relu=False --max_step=800000
```

Training Gated RVLAD (256 Clusters):

```sh
python train.py --train_data_pattern='$path_to_features/*a*??.tfrecord' --model=NetVLADModelLF --train_dir=gatedlightvladLF-256k-1024-80-0002-300iter-norelu-basic-gatedmoe --frame_features=True --feature_names="rgb,audio" --feature_sizes="1024,128" --batch_size=80 --base_learning_rate=0.0002 --netvlad_cluster_size=256 --netvlad_hidden_size=1024 --moe_l2=1e-6 --iterations=300 --learning_rate_decay=0.8 --netvlad_relu=False --gating=True --moe_prob_gating=True --lightvlad=True --max_step=600000
```

Training GRU (2 layers):

```sh
python train.py --train_data_pattern='$path_to_features/*a*??.tfrecord' --model=GruModel --train_dir=GRU-0002-1200 --frame_features=True --feature_names="rgb,audio" --feature_sizes="1024,128" --batch_size=128 --base_learning_rate=0.0002 --gru_cells=1200 --learning_rate_decay=0.9 --moe_l2=1e-6 --max_step=300000
```

Training LSTM (2 layers):

```sh
python train.py --train_data_pattern='$path_to_features/*a*??.tfrecord' --model=LstmModel --train_dir=lstm-0002-val-150-random-2 --frame_features=True --feature_names="rgb,audio" --feature_sizes="1024,128" --batch_size=128 --base_learning_rate=0.0002 --iterations=150 --lstm_random_sequence=True --max_step=400000
```


## Inference

We will write the predictions into 7 different files and then ensemble them.
Run each one of this command to run inference for a each model.

```sh
python inference.py --output_file=test-lstm-0002-val-150-random.csv --input_data_pattern='$path_to_features/test*.tfrecord' --model=LstmModel --train_dir=lstm-0002-val-150-random --frame_features=True --feature_names="rgb,audio" --feature_sizes="1024,128" --batch_size=1024 --base_learning_rate=0.0002 --iterations=150 --lstm_random_sequence=True --run_once=True --top_k=50

python inference.py --output_file=test-gatedlightvladLF-256k-1024-80-0002-300iter-norelu-basic-gatedmoe.csv --input_data_pattern='$path_to_features/test*.tfrecord' --model=NetVLADModelLF --train_dir=gatedlightvladLF-256k-1024-80-0002-300iter-norelu-basic-gatedmoe --frame_features=True --feature_names="rgb,audio" --feature_sizes="1024,128" --batch_size=1024 --base_learning_rate=0.0002 --netvlad_cluster_size=256 --netvlad_hidden_size=1024 --moe_l2=1e-6 --iterations=300 --learning_rate_decay=0.8 --netvlad_relu=False --gating=True --moe_prob_gating=True --lightvlad=True --run_once=True  --top_k=50 

python inference.py --output_file=test-gateddboflf-4096-1024-80-0002-300iter-gatedmoe.csv --input_data_pattern='$path_to_features/test*.tfrecord' --model=GatedDbofModelLF --train_dir=gateddboflf-4096-1024-80-0002-300iter-gatedmoe --frame_features=True --feature_names="rgb,audio" --feature_sizes="1024,128" --batch_size=512 --base_learning_rate=0.0002 --dbof_cluster_size=4096 --dbof_hidden_size=1024 --moe_l2=1e-6 --iterations=300 --dbof_relu=False --moe_prob_gating=True --run_once=True --top_k=50

python inference.py --output_file=test-gatednetfvLF-128k-1024-80-0002-300iter-norelu-basic-gatedmoe.csv --input_data_pattern='$path_to_features/test*.tfrecord' --model=NetFVModelLF --train_dir=gatednetfvLF-128k-1024-80-0002-300iter-norelu-basic-gatedmoe --frame_features=True --feature_names="rgb,audio" --feature_sizes="1024,128" --batch_size=1024 --base_learning_rate=0.0002 --fv_cluster_size=128 --fv_hidden_size=1024 --moe_l2=1e-6 --iterations=300 --learning_rate_decay=0.8 --fv_relu=False --gating=True --moe_prob_gating=True --fv_couple_weights=False --top_k=50

python inference.py --output_file=test-GRU-0002-1200.csv --input_data_pattern='$path_to_features/test*.tfrecord' --model=GruModel --train_dir=GRU-0002-1200 --frame_features=True --feature_names="rgb,audio" --feature_sizes="1024,128" --batch_size=1024 --base_learning_rate=0.0002 --gru_cells=1200 --learning_rate_decay=0.9 --moe_l2=1e-6 --run_once=True --top_k=50

python inference.py --output_file=test-gatednetvladLF-256k-1024-80-0002-300iter-norelu-basic-gatedmoe.csv --input_data_pattern='$path_to_features/test*.tfrecord' --model=NetVLADModelLF --train_dir=gatednetvladLF-256k-1024-80-0002-300iter-norelu-basic-gatedmoe --frame_features=True --feature_names="rgb,audio" --feature_sizes="1024,128" --batch_size=1024 --base_learning_rate=0.0002 --netvlad_cluster_size=256 --netvlad_hidden_size=1024 --moe_l2=1e-6 --iterations=300 --learning_rate_decay=0.8 --netvlad_relu=False --gating=True --moe_prob_gating=True --run_once=True  --top_k=50
```

## Averaging the models

After inference done for all the models just run:


```sh
python final_averaging.py
```

It will just take you the time to make a coffee and the submission file will be written in WILLOW_submission.csv 
as soon as you finished to drink it :D.

Antoine Miech
