# Example of building and training a tensorflow model in GCP

This project is from the GCP tutorial [Building a Recommendation System in TensorFlow](https://cloud.google.com/solutions/machine-learning/recommendation-system-tensorflow-overview), which walks the user through a simple model build script, training using CloudML, and deployment using GCP services.  I found this is be a really nice overview of how models should get packaged for training/deployment on the CloudML and GCP infrastructure.

However, there are enough bugs in the code and tutorial descriptions that I'm forking the code here to keep track of the edits and point out pitfalls.

## Part 1: Create the model
[Tutorial part 1 instruction](https://cloud.google.com/solutions/machine-learning/recommendation-system-tensorflow-create-model)  

### Notes:
* You can't run this locally on Windows.  It requires Python 2.7 and there isn't a tensorflow build for Windows that currently supports Python2.  Chalk up another one for the Mac/Linux people...I would recommend just creating a Compute Engine VM instance as described in the tutorial and using that environment for local testing.  You can train all three levels of the model described (100k, 1m, and 20m training examples) in less than ~15 minutes and the Compute Engine instance for this is neglible cost.  Just make sure you choose **v2CPU with n1-standard-2**.
* When you create a new project within GCP, you automatically get a compute engine service account (SA) and an app engine SA provisioned to you.  I think about these essentially as bot accounts that run your compute/app engines so your real user credentials/key don't have to be associated with those.  Anyway, you need to set your default SA as the SA under the Identity and API Access options in Create VM Instance, then give the VM full access to all cloud APIs.  It takes a few minutes for the default SAs to be created (like 10 or so...).
* Alternately, you could probably use the Cloud Shell to train the first two, smaller models but it may not have enough memory to train the largest model.
* After you spin up the VM, first thing is to install (sudo apt-get install {git, miniconda,bzip2,unzip}).
* Unless you have the gsutil python library and gcloud command line tool installed on your local machine, I'd recommend just leaving the VM instance you created in this step running if you plan to immediately move on to Part 2 (even though the tutorial says you don't need the VM for Part 2, its just easier since you already have the project cloned here, etc.)
* If you aren't going to use the VM immediately for the remainder of the tutorial, **remember to kill the instance before walking away.**  

## Part 2: Train and Tune on Cloud ML Engine
[Tutorial part 2 instructions](https://cloud.google.com/solutions/machine-learning/recommendation-system-tensorflow-train-cloud-ml-engine)

### Notes:
* Remember to turn on the API for CloudML engine before you get started.
* Yeah this was pretty frustrating, there are some outright wrong statements in the tutorial and bugs in the code such that nothing will run after you deploy to CloudML.
* In the first step of the tutorial when you create a Cloud Storage bucket, **be sure to put this in the same region as your VM**.
* In your VM instance (or wherever you've cloned the `tensorflow-recommendation-wals` directory, vim/nano into `wals_ml_engine/mltrain.sh` and change `BUCKET="gs://rec_serve"` (line 52) to the name of the bucket you just created.
* The code snippets given in the tutorial to deploy the training jobs don't work properly (for the second two models).  Use this:  
100k dataset: `./mltrain.sh train gs://$BUCKET/data/u.data`  
1m dataset: `./mltrain.sh train gs://$BUCKET/data/ratings.dat --delimiter ::`  
20m dataset: `./mltrain.sh train gs://$BUCKET/data/ratings.csv --delimiter , --headers`  
* Also note the code change in this repo in model.py, pd.read_csv must be changed to handle dtypes using converters and specifying engine to python, otherwise the CloudML engine errors out.
