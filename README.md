# Personal-Machine-Learning-Model
3D Facial Model Reconstruction of Myself using DFL

![harshil compare](https://github.com/AnchorBlueTop/Personal-Machine-Learning-Model/assets/98157644/36c0568d-a629-4932-8727-dd41c69037c6)

Using [DeepFaceLab](https://github.com/iperov/DeepFaceLab) I have developed a RTM (Ready to merge) 3D model reconstruction of my face - a deepfake.
What started off as something that I thought would be simple and straight forward, turned out to be quite difficult in terms of getting the predicted learned face to look as good as possible in the final video. This is a quick rundown of my many months of experience with deepfaking and what I've learnt from this process.

## Main Takeaways

* How to interpret loss values.
* Optimzing my source dataset (5000 imaages of myself)
* Make informative decisions during training that will give me the best possible output.
* Avoid overtraining.
* Machine Learning Libraries (TensorFlow, cuDNN) for Linux (Better Performance)

I have pretrained a variety of different models using the inbuilt pretrain data-set of 10000 stock images. 
Here are the main SAEHD model options/dimensions(dims) for my GPU (RTX 4090):

| Resolution  | 544x544 (Main) | 384x384 | 352x352 | 512x512                                                   
| ------------- | ------------- | ------------- | ------------- | ------------- |          
| Face Type  | WF (Whole Face)  |  WF | WF | Head |      
| Architecture  | LIAE-UDT  | LIAE-UD  | LIAE-UDT | DF-UD |
| AutoEncoder Dims  | 256  | 308  | 256 | 256 | 
| Encoder Dims  | 74  | 78  | 74 | 72 | 
| Encoder Dims  | 74  | 78  | 74 | 72 |
| Decoder Mask Dims  | 32  | 32  | 32 | 18 |
| AdaBelief Optimizer   | FALSE  | FALSE  | TRUE | TRUE |
| Batch Size   | 8  | 14  | 16  | 10 |

## Model Phases.

Every model I train can be boiled down to 3 phases:
1. Random Warp (RW) - Inital Generalization of the Face
2. Refinement with Learning Rate Dropout (LRD) - Learning the finer details of the face, Stabalize Face.
3. GAN (General Generative Adversarial Networks - Sharpen the Faces

Keep in mind, each phase also contain additional steps that will further decrease loss values before we move onto another phase.
However, once we move onto another phase, we cannot go back, or it's much better to use a backup save if the model collapses.

On average each phase takes about 30 hours to complete. 

## How to make informed decisions during training.

Knowing when to proceed to the next phase is down by looking at the face previews and by interpreting model loss values.
Model loss usually follows a logarithmic trend downwards during RW before stagnating during refinement and GAN phases.

![Untitled-4](https://github.com/AnchorBlueTop/Personal-Machine-Learning-Model/assets/98157644/341e3a21-cd55-4cda-960c-3043a56717f4)

SRC (Source Faceset) and Destination (DST) Loss values are shown here:

![Untitled-6](https://github.com/AnchorBlueTop/Personal-Machine-Learning-Model/assets/98157644/5899a485-cdb6-47b1-bed6-be0a89458dfb)

[Local Time of Save] [Iteration No.] [Iteration Time] [SRC Loss] [DST Loss]

SRC Loss measures how well the model is generating the facial features of the source face. It quantifies the difference between the generated facial features and the actual features of the source face in the training data. Lower source loss values indicate that the model is getting better at reproducing the source face.

Similarly, the destination loss measures the quality of the generated destination face. It assesses how closely the model's output matches the features of the destination face in the training data. Lower destination loss values indicate better performance in recreating the destination face. 

Essentially, once loss values no longer improve, and there isn't a notable difference in the previews after a period of time in training, then we can move onto an additional step or stop training. 

It is also important to note that overtraining is also a concern, as training for too long can cause undeseriable effects.  
Here is an example:

![annotated](https://github.com/AnchorBlueTop/Personal-Machine-Learning-Model/assets/98157644/dc03d95d-f0c4-4454-9118-89dc4b4f8d9b)

Certain settings such as Eyes & Mouth Priority (EMP) can have adverse effects on our model despite seemigly reducing DST loss values.  
It is important then to reguarly merge your model during each phase to interpret how much training your model really needs, otherwise additional training is then redundant. 

## Here are some additional previews:

![576p](https://github.com/AnchorBlueTop/Personal-Machine-Learning-Model/assets/98157644/2026d10e-061a-4c12-96eb-2ca999f0be03)
![576p harshil](https://github.com/AnchorBlueTop/Personal-Machine-Learning-Model/assets/98157644/a7bb75a2-ec37-4460-8633-c591cd110d37)

![k3g](https://github.com/AnchorBlueTop/Personal-Machine-Learning-Model/assets/98157644/0286e5b3-2d46-42f4-9c6c-e41226fb2890)

To conclude, as a hobby I make these deepfakes for fun and entertainment purposes. There is a fine line with this sort of technology and I make sure I follow all legal laws and
my own moral and ethical values. I have heard a few people tell me that they believed the deepfaked video to be the real thing, if they hadn't seen the original or knew it was fake.  So yeah I guess great power comes with great responsibility 😎







