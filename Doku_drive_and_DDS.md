# Dokumentation of "drive_and_DDS.py"

## The Imports
```
import carla 
import time 
from dataclasses import dataclass
from cyclonedds.domain import DomainParticipant
from cyclonedds.core import Qos, Policy
from cyclonedds.pub import DataWriter
from cyclonedds.sub import DataReader
from cyclonedds.topic import Topic
from cyclonedds.idl import IdlStruct
from cyclonedds.idl.annotations import key
import numpy as np
import os
```
All Carla and dds relevant resources are imported. In this script cyclone dds is used which stands for **D**ata **D**istribution **S**ervice. This allows the transfer of data between different platform like in this case from a simulated vehicle to another platform elsewhere.

```
name = f"{os.getpid()}"
obs = False
```
The name and obs variable are set depending on the id of the os (os is used in programs that communicate between platforms)


## setting up DDS
```
@dataclass
class VehicleData(IdlStruct, typename="Chatter"):
    name: str
    key("name")
    message: str
    speed: float
    obstacle: bool

rng = np.random.default_rng()
dp = DomainParticipant()
tp = Topic(dp, "Hello", VehicleData, qos=Qos(Policy.Reliability.Reliable(0)))
dw = DataWriter(dp, tp)
dr = DataReader(dp, tp)

```
The vehicle class is set up and all its descriptive parameters are set like name, key, message, speed and obstacle. Then the commuication is set up with the DataWriter and DataReader beeing set up.

```
def process_lidar_data(data):
    global obs
    point_cloud = data.raw_data
    if len(point_cloud) % 3 == 0:
        points_in_front = False
        for point_index in range(0, len(point_cloud), 3):
            if point_index + 2 < len(point_cloud):
                x = point_cloud[point_index]
                y = point_cloud[point_index + 1]
                z = point_cloud[point_index + 2]
                if 0.0 < z < 2.0 and abs(x) < 1.5:
                    points_in_front = True
                    break
        if points_in_front:
            obs = True
        else:
            obs = False
```
This is a Method to analyse the point cloud data from the CARLA lidar sensor. It checks the array of elements for objects that are within a specific range. If this is the case the obs variable will be set to true, otherwise to false.

## The lidar Sensor
```
lidar_bp = bp_lib.find('sensor.lidar.ray_cast')
lidar_transform = carla.Transform(carla.Location(x=1.0, z=2.0), carla.Rotation())
lidar_sensor = world.spawn_actor(lidar_bp, lidar_transform, attach_to=vehicle)

lidar_sensor.listen(process_lidar_data)
```
This sets up the sensore from the selection of CARLA sensors. The sensor is attached to the front of the vehicle and the listen for the sensor data method is called ".listen()"

```
spectator = world.get_spectator() 
transform = carla.Transform(vehicle.get_transform().transform(carla.Location(x=-4,z=2.5)),vehicle.get_transform().rotation) 
spectator.set_transform(transform)
vehicle.set_autopilot(True) 
```
The Spectator is defined and set to behind and slightly above the vehicle. Also the autopilot for the car is enabled.

## The Simulation Logic
```
while True:
    world.tick()
```
The Start of the Simulation Loop where the Simulation Logic is kept running with world.tick()

```
    velocity = vehicle.get_velocity()
    speed = 3.6 * (velocity.x**2 + velocity.y**2 + velocity.z**2)**0.5 
    rounded_speed = round(speed, 0)
```
The vehicle speed is calculated by taking the cars velocity and converting it into km/h

```
    sample = VehicleData(name=name, message="Current_Vehicle_Speed", speed=rounded_speed, obstacle=obs)
    print("Writing ", sample)
    dw.write(sample)
    for sample in dr.take(10):
        print("Read ", sample)
    
```
The DDS Data is read from the VehicleData class and the class variable like name or speed are print in the terminal.


```
  transform2 = carla.Transform(vehicle.get_transform().transform(carla.Location(x=-4,z=2.5)),vehicle.get_transform().rotation) 
    spectator.set_transform(transform2)
```
The specator is constantly set to directly behind the car.