The code in this repository uses Google's Machine Learning Engine to classify images. It performs three operations:
- Creates an `Experiment` that launches a `DNNClassifier`
- Trains the classifier with the image data from mnist_train.tfrecords
- Tests the classifier with the image data from mnist_test.tfrecords

Instead of performing these operations on the development system, the module and the image files need to be uploaded to Google Cloud Storage (GCS). Then the training job can be submitted with a `gcloud` command like the following:

```
gcloud ml-engine jobs submit training cloud_mnist --module-name trainer.task --package-path trainer --job-dir JOB_DIR --region REGION 
--staging-bucket STAGING_BUCKET --stream-logs -- --data_dir DATA_DIR
```

This tells the engine that the application code is provided in a package named `trainer`, and the name of the module is `task`. The package is located in the GCS bucket given by `JOB_DIR` and the image data is stored in the GCS bucket named `DATA_DIR`.

For the engine to recognize the package, the package directory must meet three requirements:
- The directory must contain the module identified by `--module-name`
- The parent directory must have a file named setup.py
- Every directory under the parent directory must have a file named __init__.py

The engine will install the package if setup.py performs two operations: 
- Import `setuptools.setup`
- Call the `setup` function of the `setuptools` module
  
The setup function accepts a great deal of information about the package, including its name, version, and dependencies. The following code shows how it can be called:

```
REQUIRED_PACKAGES = ['tensorflow>=1.8']
setup(
    name='trainer',
    version='0.1',
    install_requires=REQUIRED_PACKAGES,
    packages=find_packages(),    
    include_package_data=True,    
    author='Matthew Scarpino'
    description='Running MNIST classification in the cloud'
)
```

As shown, `setup` tells the engine that the application requires TensorFlow. In this case, the application requires a version of TensorFlow greater than 1.8.
