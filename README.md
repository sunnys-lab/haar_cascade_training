# Train your own OpenCV Haar classifier in Windows OS

**Important**: This guide assumes you work with OpenCV 2.4.x.

This repository aims to provide tools and information on training your own
OpenCV Haar classifier.  Use it in conjunction with this blog post: [Train your own OpenCV Haar
classifier](http://coding-robin.de/2013/07/22/train-your-own-opencv-haar-classifier.html).


## Instructions
1. get perl and OpenCV for winodws & install

  perl: http://strawberryperl.com/download/5.28.1.1/strawberry-perl-5.28.1.1-64bit.msi
  
  OpenCV: https://sourceforge.net/projects/opencvlibrary/files/opencv-win/2.4.13/opencv-2.4.13.6-vc14.exe/download

2. Clone this repository

   git clone https://github.com/sunnys-lab/haar_cascade_training.git

3. Put your positive images in the `./pos` folder

4. Put the negative images in the `./neg` folder 

5. Create a list of them using 'create_pos_n_neg()' function in 'download.py'

5. Create positive samples with the `bin/createsamples.pl` script and save them to the `./samples` folder:

        perl bin/createsamples.pl positives.txt negatives.txt samples 1500 "opencv_createsamples -bgcolor 0 -bgthresh 0 -maxxangle 1.1 -maxyangle 1.1 maxzangle 0.5 -maxidev 40 -w 80 -h 40"

6. Use `mergevec.py` to merge the samples in `./samples` into one file:

        python ./tools/mergevec.py -v samples/ -o samples.vec

   Note: If you get the error `struct.error: unpack requires a string argument of length 12`
   then go into your **samples** directory and delete all files of length 0.

7. Start training the classifier with `opencv_traincascade`, which comes with OpenCV, and save the results to `./classifier`:

        opencv_traincascade -data classifier -vec samples.vec -bg negatives.txt -numStages 20 -minHitRate 0.999 -maxFalseAlarmRate 0.5 -numPos 1000 -numNeg 600 -w 80 -h 40 -mode ALL -precalcValBufSize 1024 -precalcIdxBufSize 1024
          
    If you want to train it faster, configure feature type option with LBP:

        opencv_traincascade -data classifier -vec samples.vec -bg negatives.txt -numStages 20 -minHitRate 0.999 -maxFalseAlarmRate 0.5 -numPos 1000 -numNeg 600 -w 80 -h 40 -mode ALL -precalcValBufSize 1024 -precalcIdxBufSize 1024 -featureType LBP

    After starting the training program it will print back its parameters and then start training. Each stage will print out some analysis as it is trained:

      ```
      ===== TRAINING 0-stage =====
      <BEGIN
      POS count : consumed   1000 : 1000
      NEG count : acceptanceRatio    600 : 1
      Precalculation time: 11
      +----+---------+---------+
      |  N |    HR   |    FA   |
      +----+---------+---------+
      |   1|        1|        1|
      +----+---------+---------+
      |   2|        1|        1|
      +----+---------+---------+
      |   3|        1|        1|
      +----+---------+---------+
      |   4|        1|        1|
      +----+---------+---------+
      |   5|        1|        1|
      +----+---------+---------+
      |   6|        1|        1|
      +----+---------+---------+
      |   7|        1| 0.711667|
      +----+---------+---------+
      |   8|        1|     0.54|
      +----+---------+---------+
      |   9|        1|    0.305|
      +----+---------+---------+
      END>
      Training until now has taken 0 days 3 hours 19 minutes 16 seconds.
      ```

    Each row represents a feature that is being trained and contains some output about its HitRatio and FalseAlarm ratio. If a training stage only selects a few features (e.g. N = 2) then its possible something is wrong with your training data.

    At the end of each stage the classifier is saved to a file and the process can be stopped and restarted. This is useful if you are tweaking a machine/settings to optimize training speed.

8. Wait until the process is finished (which takes a long time — a couple of days probably, depending on the computer you have and how big your images are).

9. Use your finished classifier!



## References & Links:

- [Naotoshi Seo - Tutorial: OpenCV haartraining (Rapid Object Detection With A Cascade of Boosted Classifiers Based on Haar-like Features)](http://note.sonots.com/SciSoftware/haartraining.html)
- [Material for Naotoshi Seo's tutorial](https://code.google.com/p/tutorial-haartraining/)
- [OpenCV Documentation - Cascade Classifier Training](http://docs.opencv.org/doc/user_guide/ug_traincascade.html)
