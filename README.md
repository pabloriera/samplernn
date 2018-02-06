## SampleRNN for speech synthesys

Keras implementation of SampleRNN model published [here](https://arxiv.org/abs/1612.07837).
This repo does only three tier architecture ![model](/model3t.png). Original audio sequence is feed
to 3 inputs. Input_1(on picture) goes to slow tier RNN that groups 8 audio samples into 1 timestep.
Mid tier gets 2 audio samples at the time plus input from slow tier(see add_1). Finally the samples
are being generated by MLP that gets embedding of the previos audio sample(input_3) and output from mid
tier layer(see add_2)

## Audio preprocessing
Before we can start training audio must undergo some preprocessing. The process to follow is:
* `mkdir -p blizzard/tiny`
* copy some wav files to ./blizzard/tiny; for example 1 min of audio in total
* run `python preprocess.py $PWD/blizzard/tiny`
* blizzard/tiny_parts now contains audio material split into 8seconds long chunks

## Baseline
 [Original implementation of the SampleRNN can be found here.](https://github.com/soroushmehr/sampleRNN_ICLR2017)
 It served as baseline reference during the development. Training 
 results on 'tiny' (see below) dataset were compared with the baseline.
 Below the costs in bits per sequence for this code and baseline are shown
 
 *This code* epoch | Training | Validation
 ---|---| ---
 1 | 3.98438 | 4.87372
 10 | 2.29819 | 4.14896
 
 *Baseline* epoch | Training | Validation
 ---|---| ---
 1 | 3.9624 | 4.9070
 10 | 2.6645 | 4.2562
 

## Training
Unfortunately start/stop indexes to separate validation and training data sets are to be picked manually.
Depending on the dataset size. Following values were used for 2 datasets namely tiny and blizzard2013.
Index of last training sequence is given by **--trainstop** command arg(see below) and **--validstop**
points to the last validation sequence index.

Dataset | --trainstop | --validstop | minibatch size
--- | --- | --- | ---
tiny(~50sec) | 4 | 6 | 2
blizzard2013(~20h) | 8000 | 9000 | 100

To start training run `THEANO_FLAGS=device=cpu,mode=FAST_RUN python train_srnn.py --exp=tiny --slowdim=32  --dim=32 --cutlen=512 --batchsize=2 --validstop=6 --trainstop=4`. This will create model with 32 hidden units in each layer and
run tbpp for 512 timestamps (due to --cutlen=512). Using theano backend and CPU to compute.

After about 3 epochs on blizzard2013 dataset model should be able to generate nice looking and
even sounding samples. Here how it looks after nearly one epoch of 12 hours of free GPU at [Colab](https://colab.research.google.com) ![sample](/sample4s.png)
The audio sample shown on the picture can be found in [sample4s.wav](/sample4s.wav)


## Sampling

Training process produces files named `<tiny|all>_srnn_sz<dim>_e<epoch>.h5` with model weights every *--svepoch* and in the end of the training. Choose the one with the best validation performance to generate a wav sample. For example
`THEANO_FLAGS=device=cpu,mode=FAST_RUN python train_srnn.py --exp=tiny --slowdim=32  --dim=32 --cutlen=512 --batchsize=2 --validstop=6 --trainstop=4 --sample=<filename>` will produce *generated.wav*

## Sampling from pretrained model
This repo contains a [file allmost_1e.h5](/allmost_1e.h5) with model weights after about 12 hours of training on K80 GPU using  about 20 hours of audio from Blizzard2013. Thus it is possible to try it right away and do audio sampling using following command `THEANO_FLAGS=device=cpu,mode=FAST_RUN python train_srnn.py --slowdim=1024  --dim=1024  --sample=allmost_1e.h5`. Which will use CPU and Theano backend to do the work




