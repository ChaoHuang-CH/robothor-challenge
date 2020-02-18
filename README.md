<p align="center">
  <img width = "50%" src='/images/robothor_challenge_logo.svg' />
  </p>

--------------------------------------------------------------------------------

# RoboTHOR-Challenge

Welcome to the RoboTHOR Challenge. The task for the RoboTHOR Challenge is to build a model/agent that can navigate towards a particular object in a room using the [RoboTHOR](https://ai2thor.allenai.org) embodied agent environment. Please follow the instructions below to get started.

## Installation

To begin working on your own model you must have Docker installed on your host and a Nvidia GPU (required for 3D rendering).


Clone or fork this repository
```bash
git clone https://github.com/allenai/robothor-challenge
```

Build the Docker container
```
cd robothor-challenge
./scripts/build.sh
```

Run evaluation on random agent
```
./scripts/evaluate_train.sh
```

At this point you should see log messages that resemble the following:
```
2020-02-11 05:08:00,545 [INFO] robothor_challenge - Task Start id:59 scene:FloorPlan_Train1_1 target_object:BaseballBat|+04.00|+00.04|-04.77 initial_position:{'x': 7.25, 'y': 0.910344243, 'z': -4.708334} rotation:180
2020-02-11 05:08:00,895 [INFO] robothor_challenge - Agent action: MoveAhead
2020-02-11 05:08:00,928 [INFO] robothor_challenge - Agent action: RotateLeft
2020-02-11 05:08:00,961 [INFO] robothor_challenge - Agent action: MoveBack
2020-02-11 05:08:00,989 [INFO] robothor_challenge - Agent action: Stop
```


## Model

Your model must subclass ```robothor_challenge.agent.Agent``` and implement the method ```on_event```. For an episode to be successful, the agent must be within 1 meter of the target object and the object must also be visible to the agent.  To declare success, respond with the ```Stop``` action.  If ```Stop``` is not sent within the maxmimum number of steps (100 max), the episode will be considered failed and the next episode will be initialized.  The following agent (found in example_agent.py) takes a random action on each event:

```python
from robothor_challenge.agent import Agent
from robothor_challenge import RobothorChallenge
import random
import logging
logging.getLogger().setLevel(logging.INFO)


class SimpleRandomAgent(Agent):

    def on_event(self, event):
        action = random.choice(['MoveAhead', 'MoveBack', 'RotateRight', 'RotateLeft', 'LookUp', 'LookDown', 'Stop'])
        return action

if __name__ == '__main__':
    r = RobothorChallenge(agent_cls=SimpleRandomAgent)
    r.inference()
```

The agent will have access to the episode as a member variable ```agent.episode```.  The structure of each episode is as follows:
```javascript
 {
        "difficulty": "easy", // Task difficulty
        "id": 0,
        "initial_orientation": 180, // Initial orientation of the agent
        "initial_position": { // Initial position of the agent
            "x": 3.0,
            "y": 0.910344243,
            "z": -1.75
        },
        "object_id": "Apple|+01.98|+00.77|-01.75", // Id of the target object
        "object_type": "Apple", // Target object category
        "scene": "FloorPlan_Train1_1", // Name of the scene
        "shortest_path": [ // Coordinates of the points along the shortest path
            {
                "x": 3.0,
                "y": 0.0103442669,
                "z": -1.75
            },
            {
                "x": 2.75,
                "y": 0.0103442669,
                "z": -1.75
            }
        ],
        "shortest_path_length": 0.25, // Length of the shortest path
        "target_position": { // Position of the target
            "x": 1.979,
            "y": 0.7714,
            "z": -1.753
        }
}

```

When you have a model to evaluate, modify the Dockerfile to copy any files and update the requirements.txt as needed. Once you have built the image, you can run the evaluation script to calculate the SPL value.

## Dataset

The dataset consists of 25074 training episodes and 6290 val episodes where each episode consists of an agent starting position/rotation and target object.  

The following target object types exist in the dataset:
* Alarm Clock
* Apple
* Baseball Bat
* Basketball
* Garbage Can
* House Plant
* Laptop
* Mug
* Spray Bottle
* Television
* Vase

| Split | Difficulty | Total |
| ----- |:----------:|:-----:|
|Train|easy| 8125 | 
|Train|medium| 8125 | 
|Train|hard| 8824| 
|Val|easy| 2038 | 
|Val|medium| 2038 | 
|Val|hard| 2214| 



All the episodes for each split (train/val) can be found within the dataset/{train/val}.json files.  Configuration parameters for the environment can be found within dataset/challenge_config.yaml.  These are the same values that will be used for generating the leaderboard.  You are free to train your model with whatever parameters you choose, but these params will be reset to the original values for leaderboard evaluation.

## Challenge Submissions

We will be using [EvalAI](https://evalai.cloudcv.org) to host the challenge.  The first phase of the challenge will begin on Feb 11, 2020. You will be submitting your docker image for evaluation using the EvalAI CLI.  During leaderboard evaluation, separate scenes/points will be used to determine your score.


## Acknowledgments

We would like to thank [Rishabh Jain](https://rishabhjain.xyz/) for the help for setting up the EvalAI leaderboard.

