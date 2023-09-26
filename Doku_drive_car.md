# Dokumentation of "drive_car.py"

## General Setup
```
import carla 
```
To run Carla you only need to import one module

```
client = carla.Client('localhost', 2000) 
world = client.get_world()
spectator = world.get_spectator() 
bp_lib = world.get_blueprint_library()  
spawn_points = world.get_map().get_spawn_points() 
```
If Carla is set up on localhost, the default port is *2000*.
The Carla World, the Spectator and the Blueprint library need to be instantiated.

```
for v in world.get_actors().filter('*vehicle*'): 
    v.destroy()
```
For this code, all actors are filtered for vehicles and they are removed so you have a clean Map with no Car Actors.

```
vehicle_bp = bp_lib.find('vehicle.lincoln.mkz_2020') 
vehicle = world.try_spawn_actor(vehicle_bp, spawn_points[13])
```
An actor vehicle is chosen from the blueprint library and created in the world at a specific spawnpoint.

```
vehicle.set_transform(carla.Transform(carla.Location(x=7.561335, y=111.042351, z=0.931391), carla.Rotation(pitch=8.095427, yaw=-4.816957, roll=0.000086)))
```
The vehicle is set to a starting position

```
spec_transform = carla.Transform(carla.Location(x=15.012684, y=150.451126, z=52.484871), carla.Rotation(pitch=-32.483482, yaw=-89.044197, roll=0.000016))
spectator.set_transform(spec_transform)
```
The Spectator is moved to a position where the Viewer can observe the vehicle better. Positions in CARLA are made of *x*, *y* and *z* axis plus the vehicle rotation in itself (*pitch*, *yaw*, *roll*)

## The Routeplaner

```
town_map = world.get_map()
roads = town_map.get_topology()
vehicle_location = vehicle.get_location()
```
The map is instantiated from the variable world, the variable roads is instantiated from map variable. Also the vehicle current position is stored.

```
import sys
sys.path.append('/home/ubuntu/CARLA_0.9.14/PythonAPI/carla') 
from agents.navigation.global_route_planner import GlobalRoutePlanner
from agents.navigation.basic_agent import BasicAgent
```
Then, everything you need for the Routeplaner is imported. The Routeplaner works with an Agent that plots the route and the drives the car the given Route.

```
sampling_resolution = 2
grp = GlobalRoutePlanner(town_map, sampling_resolution)
```
The RoutePlaner is set up with the map and its sampling_resolution.

```
point_a = vehicle_location
point_b = carla.Location(x=81.05, y=33.19, z=1.85)
route = grp.trace_route(point_a,point_b)
```
After the starting and end points are set as variables, the route is calculated from the RoutePlaner

```
for waypoint in route :
    world.debug.draw_string(waypoint[0].transform.location, '^', draw_shadow=False, 
        color=carla.Color(r=0, g=0,b=255), life_time=120,
        persistent_lines=True)
```
In a *For Loop* every Waypoint on the Route gets a marked dot in the shape of a *^* in the Color Blue which remains there for 120 Seconds.

```
agent = BasicAgent(vehicle)
agent.set_destination(point_b)
agent.set_target_speed(30) #set The maximum speed of the car
```
The Agent is instantiated on the vehicle variable, which represents the car Actor. The destination and the vehicle speed is set for the Actor.

```
while True:
    if agent.done():
        print('Target Location reached!')
        break
    vehicle.apply_control(agent.run_step())
```
In this *while Loop* the agent moves the car continuously until the destination (*point_b*) is reached. The it will print the Message "*Target Location reached!*" and the car will stop.