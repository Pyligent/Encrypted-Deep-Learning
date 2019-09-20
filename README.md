# Encrypted Deep Learning

## Step 1: Public Training

By private predictions, we mean that the data is constantly encrypted throughout the entire process. At no point is the user sharing raw data, only encrypted (that is, secret shared) data. In order to provide these private predictions, Syft Keras uses a library called [TF Encrypted](https://github.com/tf-encrypted/tf-encrypted) under the hood. TF Encrypted combines cutting-edge cryptographic and machine learning techniques, but you don't have to worry about this and can focus on your machine learning application.
Start serving private predictions with only three steps:
- **Step 1**: train your model with normal Keras.
- **Step 2**: secure and serve your machine learning model (server).
- **Step 3**: query the secured model to receive private predictions (client). 

Alright, let's go through these three steps so you can deploy impactful machine learning services without sacrificing user privacy or model security.


## Train Your Model in Keras

To use privacy-preserving machine learning techniques for your projects you should not have to learn a new machine learning framework. If you have basic [Keras](https://keras.io/) knowledge, you can start using these techniques with Syft Keras. If you have never used Keras before, you can learn a bit more about it through the [Keras documentation](https://keras.io). 

Before serving private predictions, the first step is to train your model with normal Keras. As an example, we will train a model to classify handwritten digits. To train this model we will use the canonical [MNIST dataset](http://yann.lecun.com/exdb/mnist/).

We borrow [this example](https://github.com/keras-team/keras/blob/master/examples/mnist_cnn.py) from the reference Keras repository.  To train your classification model, you just run the cell below.


## Step 2: Load and Serve the Model
Now that you have a trained model with normal Keras, you are ready to serve some private predictions. We can do that using Syft Keras.

To secure and serve this model, we will need three TFEWorkers (servers). This is because TF Encrypted under the hood uses an encryption technique called [multi-party computation (MPC)](https://en.wikipedia.org/wiki/Secure_multi-party_computation). The idea is to split the model weights and input data into shares, then send a share of each value to the different servers. The key property is that if you look at the share on one server, it reveals nothing about the original value (input data or model weights).

We'll define a Syft Keras model like we did in the previous notebook. However, there is a trick: before instantiating this model, we'll run `hook = sy.KerasHook(tf.keras)`. This will add three important new methods to the Keras Sequential class:
 - `share`: will secure your model via secret sharing; by default, it will use the SecureNN protocol from TF Encrypted to secret share your model between each of the three TFEWorkers. Most importantly, this will add the capability of providing predictions on encrypted data.
 - `serve`: this function will launch a serving queue, so that the TFEWorkers can can accept prediction requests on the secured model from external clients.
 - `shutdown_workers`: once you are done providing private predictions, you can shut down your model by running this function. It will direct you to shutdown the server processes manually if you've opted to manually manage each worker.
 
 ## Step 3: Setup Your Worker Connectors

Let's now connect to the TFEWorkers (`alice`, `bob`, and `carol`) required by TF Encrypted to perform private predictions. For each TFEWorker, you just have to specify a host.

These workers run a [TensorFlow server](https://www.tensorflow.org/api_docs/python/tf/distribute/Server), which you can either manage manually (`AUTO = False`) or ask the workers to manage for you (`AUTO = True`). If choosing to manually manage them, you will be instructed to execute a terminal command on each worker's host device after calling `model.share()` below.  If all workers are hosted on a single device (e.g. `localhost`), you can choose to have Syft automatically manage the worker's TensorFlow server.


## Step 4: Split the Model Into Shares

Thanks to `sy.KerasHook(tf.keras)` you can call the `share` method to transform your model into a TF Encrypted Keras model.

If you have asked to manually manage servers above then this step will not complete until they have all been launched. Note that your firewall may ask for Python to accept incoming connection.

## Step 5: Launch 3 Servers

```
python -m tf_encrypted.player --config /tmp/tfe.config server0
python -m tf_encrypted.player --config /tmp/tfe.config server1
python -m tf_encrypted.player --config /tmp/tfe.config server2```


## Step 6: Serve the Model

Perfect! Now by calling `model.serve`, your model is ready to provide some private predictions. You can set `num_requests` to set a limit on the number of predictions requests served by the model; if not specified then the model will be served until interrupted.


## Step 7: Run the Client

At this point open up and run the companion notebook: Section 4b - Encrytped Keras Client


## Step 8: Shutdown the Servers

Once your request limit above, the model will no longer be available for serving requests, but it's still secret shared between the three workers above. You can kill the workers by executing the cell below.

**Congratulations** : Secure Classification with Syft Keras and TFE!
