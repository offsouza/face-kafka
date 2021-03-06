# face-kafka

## Project Title
This project is about detection and recognition of faces in real time with Apache Kafka message passing architecture.
Ideally,there can be multiple remote video streams captured from multiple IP cams ,but since this is a prototype version, we're using a single video source (local or remote), a single Kafka broker , producer and consumer.
But, this prototype application can be scaled up for multiple video sources with some changes.

Therefore a producer captures the data frame by frame from the video source, serializes it to a bytestring, and publishes it to a topic in the Kafka broker. 
The consumer then subscribes to the topic , gets the data (the bytestring of each frame) , deserializes it into a numpy array,and pushes it into the face recognition pipeline.
The frame that gets out of the face recognition pipeline, is basically the original frame with bboxes around the faces detected, with their corresponding names if the algorithm could reognize the face , else it's labelled as "Unknown".

A Flask Webserver renders these "transformed" frames on localhost:5000.

On a sidenote, the Face detection algorithm can either be dlib HOG or dlib CNN detectors , which can be provided as parameters.
And, the Face recognition algorithm is based on the Google FaceNet architecture.


## Getting Started

These instructions will get you a copy of the project up and running on your local machine for development and testing purposes.



### Prerequisites

What things you need to install the software and how to install them

- Make sure you have Apache Kafka and Anaconda installed properly on your system
- The other dependencies can be installed as follows:
  ```
  $ conda env create -f environment.yml
  ```

### Installing

A step by step series of examples that tell you how to get a development env running

```
$ git clone https://github.com/rtb7syl/face-kafka.git
$ cd face-kafka
$ source activate face-kafka
```

### Get it running

Now that all the dependencies are installed, and the environment is activated, it's time to run the app.

First things first, we need to store the known face images somewhere.So, from the project root, type in the commands
```
$ mkdir imgs
$ cd imgs
$ mkdir known_faces
```
Inside known_faces create a directory for each person you want to identify. In each such directory corresponding to a person, store atleast 10 images of that person's face.

The directory structure will be like this:
```
face-kafka
  ..imgs
    ..known_faces
      ..person1
      ..person2
      .........
      .........
```
Now, we need to convert those images into a 128 dims vector embedding.For that, go to project root and run the command
```
$ python -m face_rec.encode_faces
```
An embeddings.pickle file will be generated in face-kafka/face_rec after this. 

Now that the face images are embedded, we can start detecting and recognizing faces from video streams.For that,
First start zookeeper and kafka broker servers in two different terminal instances

Start zookeeper server 
```
$ cd <kafka-installation-path>
$ bin/zookeeper-server-start.sh config/zookeeper.properties
```
Start Kafka server
```
$ cd <kafka-installation-path>
$ bin/kafka-server-start.sh config/server.properties
```
Now, run the producer code from another terminal instance. Go to project root.
```
$ python producer.py --source <video_source>
or
$ python producer.py -s <video_source>
```
<video_source> can be a remote or a local video stream source

Finally, run the consumer code from project root.
```
python consumer.py
```
Then go to localhost:5000 to see the detected and recognised faces video stream

### Known Issues

As of now , this project is comaptible for a Linux environment.
Since this a prototype version, the entire app runs on a local dev environment
To change number of partitions, topics etc, please feel free to play with the code.

### License
This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details
