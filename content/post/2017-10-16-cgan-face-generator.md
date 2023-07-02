---
title: "Journey of building a face generator"
date: 2017-10-16
---

![demo](https://raw.githubusercontent.com/hiepph/cgan-face-generator/master/demo/train.gif)

[Github](https://github.com/hiepph/cgan-face-generator)

I was impressed by [Image-to-Image Demo](https://affinelayer.com/pixsrv/) with pix2pix (a.k.a cGAN) model ([Paper](https://arxiv.org/abs/1611.07004)).
I and my friends tried to build one, everything from scratch in an AI hackathon.
The idea is the machine tries to generate a real face from user's sketch.

![overview](https://github.com/hiepph/cgan-face-generator/raw/master/demo/overview.png)


## Collect data

![CAF](http://i.imgur.com/qJbd8Ig.jpg)

The ultra important job is always to get yourself a good quantity and quality dataset.
This is the most-enlightened lesson I've ever learned in deep learning research.

> A Good Quality Model Begins with a Clean Dataset.
> [Source](https://makegirlsmoe.github.io/main/2017/08/14/news-english.html)

I  took a look at [CelebA](http://mmlab.ie.cuhk.edu.hk/projects/CelebA.html) dataset.
But I and my team all decided to have only girl images due to personal favorite.
So we crawled actress' images from Google, used [Dlib](http://dlib.net/) to detect & crop their faces, used [OpenCV](https://opencv.org/) to get sketches and made our own 8303 faces dataset.


Things would be perfect without noises of background.
But we couldn't find any better source or asked for more.


## Model implementation

I was lucky.

I found a [PyTorch](http://pytorch.org/) port implementation of cGAN model from [Jun-Yan Zhu](https://github.com/junyanz) (one of the authors of the original Paper) and his project [pytorch-CycleGAN-and-pix2pix](https://github.com/junyanz/pytorch-CycleGAN-and-pix2pix).
Everything is explained from how to prepare a dataset, preprocess input, train model to visualize the result.

Then the path was clear.
I already had a dataset.
I just needed to divide it into 80/20 ratio as train and validate parts, preprocess using provided script, train the model, supervise the loss and frequently check generated images each epoch.


## Training

![train](https://github.com/hiepph/cgan-face-generator/raw/master/demo/train.png)

This is the most stressed part I've ever experienced.

Train GAN is always expensive and time-consuming.
It's about training 2 networks: D for Discriminator, and G for Generator, beside this is an image-related task.

The default train script is 200 epochs, but I decided to shorten down to 30 epochs for an experiment, and due to time-limit of the hackathon.

First I try with local machine and a GPU Nvidia GeForce GTX 960, approximately 30 minutes for each epoch.
Which means 15 hours of uptime. It ate all resources of the machine, and the heat from my computer made me hard to develop other features.

So I stopped at 5th epoch, and switched to a kindly gifted AWS `p2.8xlarge` instance.
Now with 4 GPUs with 4x performance and more memory so I can fit a larger batch size, each epoch took just about 7~8 minutes to complete. Which means more 4 times faster, and it took about 3 hours to complete.

**Disclosure**: At the hackathon, the ratio was actually 95/5 for train and validation task as we tried to mirror the #edges2shoes example.
And about 25th~30th epoch, model seemed to go overfit, the loss didn't go down anymore.
Train sketch was way more beautiful than test sketch.
So after the competition, I tested again as 80/20 ratio, and the loss looked like it can go down if I continued to train more. It still took 10 long-lasting hours to complete 30 epochs.


## Model integration

Firstly, we write a simple server for uploading file task, using [Flask](http://flask.pocoo.org/):

```python
import os
import time

from flask import Flask, jsonify, request, redirect, url_for, send_file, make_response
from flask_cors import CORS, cross_origin

# Config
app = Flask(__name__)
CORS(app)
app.config['UPLOAD_FOLDER'] = os.path.join(os.getcwd(), 'upload')
app.config['ALLOWED_EXTENSIONS'] = set(['jpg', 'jpeg', 'png'])


# Helpers
def allowed_file(filename):
    return '.' in filename and \
        filename.rsplit('.', 1)[1] in app.config['ALLOWED_EXTENSIONS']

def error(msg):
    return jsonify({'error': msg})


# Routers
@app.route('/')
def pong():
    return 'Pong', {'Content-Type': 'text-plain; charset=utf-8'}


@app.route('/upload', methods=['POST'])
def upload():
    if 'file' not in request.files:
        return error('file form-data not existed'), 412

    image = request.files['file']
    if not allowed_file(image.filename):
        return error('Only supported %s' % app.config['ALLOWED_EXTENSIONS']), 415

    # Submit taylor.jpg ---> taylor_1234567.jpg (name + timestamp)
    image_name, ext = image.filename.rsplit('.', 1)
    image_name = image_name + '_' + str(int(time.time())) + '.' + ext
    # Save image to /upload
    image_path = os.path.join(app.config['UPLOAD_FOLDER'], image_name)
    image.save(image_path)

    return send_file(image_path)


if __name__ == '__main__':
    app.run(debug=True)
```

Simple enough, `GET /` for checking connection, and `POST /upload` for storing upload image.

Next is the most important part: a generated script for using trained Generator (G) model to output a fake photo from input image.
Follow this [issue](https://github.com/junyanz/pytorch-CycleGAN-and-pix2pix/issues/25), problem solved:

```python
import os

from options.test_options import TestOptions
from data.data_loader import CreateDataLoader
from models.models import create_model
import util.util as util


if __name__ == "__main__":
    if not os.path.exists('results'):
        os.mkdir('results')

    opt = TestOptions().parse()
    opt.nThreads = 1
    opt.batchSize = 1
    opt.serial_batches = True
    opt.no_flip = True

    data_loader = CreateDataLoader(opt)
    dataset = data_loader.load_data()
    model = create_model(opt)

    for i, data in enumerate(dataset):
        if i >= opt.how_many:
            break
        model.set_input(data)
        # Forward
        model.test()

        # Convert image to numpy array
        fake = util.tensor2im(model.fake_B.data)
        # Save image
        img_path = model.get_image_paths()
        img_name = img_path[0].split('/')[-1]
        util.save_image(fake, 'results/{}'.format(img_name))
```

The last part is more simple now, just fuse these above scripts:

```python
import os
import time

from flask import Flask, jsonify, request, redirect, url_for, send_file, make_response
from flask_cors import CORS, cross_origin

from options.test_options import TestOptions
from data.data_loader import CreateDataLoader
from models.models import create_model
import util.util as util


# Model loader
opt = TestOptions().parse()
opt.nThreads = 1
opt.batchSize = 1
opt.serial_batches = True
opt.no_flip = True

model = create_model(opt)


# Config
app = Flask(__name__)
CORS(app)
app.config['UPLOAD_DIR'] = os.path.join(os.getcwd(), 'upload')
app.config['RESULT_DIR'] = os.path.join(os.getcwd(), 'results')
app.config['ALLOWED_EXTENSIONS'] = set(['jpg', 'jpeg', 'png'])

# Setup
if not os.path.exists(app.config['UPLOAD_DIR']):
    os.mkdir(app.config['UPLOAD_DIR'])

if not os.path.exists(app.config['RESULT_DIR']):
    os.mkdir(app.config['RESULT_DIR'])


# Helpers
def allowed_file(filename):
    return '.' in filename and \
        filename.rsplit('.', 1)[1] in app.config['ALLOWED_EXTENSIONS']

def error(msg):
    return jsonify({'error': msg})


# Routers
@app.route('/')
def pong():
    return 'Pong', {'Content-Type': 'text-plain; charset=utf-8'}


@app.route('/gen', methods=['POST'])
def gen():
    if 'file' not in request.files:
        return error('file form-data not existed'), 412

    image = request.files['file']
    if not allowed_file(image.filename):
        return error('Only supported %s' % app.config['ALLOWED_EXTENSIONS']), 415

    # Submit taylor.jpg ---> save image to upload/12345678/taylor.jpg (upload/timestamp/imagename.ext)
    t = int(time.time())
    image_dir = os.path.join(app.config['UPLOAD_DIR'], str(t))
    image_path = os.path.join(image_dir, image.filename)

    os.mkdir(image_dir)
    image.save(image_path)

    # Prepare data loader
    opt.dataroot = image_dir
    data_loader = CreateDataLoader(opt)
    dataset = data_loader.load_data()

    for i, data in enumerate(dataset):
        if i >= opt.how_many:
            break
        model.set_input(data)
        # Forward
        model.test()

        # Convert image to numpy array
        fake = util.tensor2im(model.fake_B.data)
        # Save image
        result_dir = os.path.join(app.config['RESULT_DIR'], str(t))
        result_path = os.path.join(result_dir, image.filename)
        os.mkdir(result_dir)
        util.save_image(fake, result_path)

    return send_file(result_path)


if __name__ == '__main__':
    app.run()
```

Now the main route is `POST /gen`, just request an image as `form-data` with `file` key, and receive response.

Some demo I achieved from a remake after the hackathon. First, let's see this trained input: generated result is beautiful and artistic.

![train](https://github.com/hiepph/cgan-face-generator/raw/master/demo/gal.png)

A test I randomly picked from the internet: a little bit ugly but the fun part is model knows some characteristics from the face like skin and hair color.
It's like a colored version of the original image.

![test](https://github.com/hiepph/cgan-face-generator/raw/master/demo/wonder.png)


That's it. For more technical detail, you can check out my main project on Github: [cgan-face-generator](https://github.com/hiepph/cgan-face-generator).
Although this is just a Back End part, build a top Front End application or web page is just a matter of time.


## Conclusion

Did I do a great job? Maybe no. All the credits go to the authors of the paper.
And this is just a small process of real-world deep learning engineering. There are so many problems if you want to bring a new idea into production and scale to reality. Just take a look at [Michelangelo](https://eng.uber.com/michelangelo/) or [TFX](http://www.kdd.org/kdd2017/papers/view/tfx-a-tensorflow-based-production-scale-machine-learning-platform).

But this project was fun. It satisfied my curiosity.
Today I learned to break down a problem to small tasks that you can solve, and focus to clear each task even if you're not so sure about it. Just take little steps, at least it'll lead you to somewhere forward.

Happy deep learning.
