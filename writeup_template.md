## Project: 3D Motion Planning
![Quad Image](./misc/enroute.png)

---


# Required Steps for a Passing Submission:
1. Load the 2.5D map in the colliders.csv file describing the environment.
2. Discretize the environment into a grid or graph representation.
3. Define the start and goal locations.
4. Perform a search using A* or other search algorithm.
5. Use a collinearity test or ray tracing method (like Bresenham) to remove unnecessary waypoints.
6. Return waypoints in local ECEF coordinates (format for `self.all_waypoints` is [N, E, altitude, heading], where the drone’s start location corresponds to [0, 0, 0, 0].
7. Write it up.
8. Congratulations!  Your Done!

## [Rubric](https://review.udacity.com/#!/rubrics/1534/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  

You're reading it! Below I describe how I addressed each rubric point and where in my code each point is handled.

### Explain the Starter Code

#### 1. Explain the functionality of what's provided in `motion_planning.py` and `planning_utils.py`
These scripts contain a basic planning implementation that includes...

`motion_planning.py:`
- A new PLANNING state has been added to the list of states
- The planing state is invoked right-after arm and before take-off and PLANNING steps are implemented in plan_path method
- In plan_path:
    - We are supposed to load and set home position for the drone.
    - A list of colliders from a csv file.
    - The create_grid method implemented in planning_utils.py creates a grid that takes into account the list of colliders and marks accessible cells
    - start position and target goal are defined and a path is computed using a_star search implemented in planning_utils.py
    - The generated path is converted to waypoints that are then set to the simulator for execution.
    - The state is transition into take-off and waypoints are traversed one by one.
    - Once goal is reached then land command is invoked followed by a disarm.
    
`planning_utils.py:`
- Contains create_grid explained earlier.
- A class the defines all possible Action s and their individual cost.
- A method to identify the list of valid actions at a certain node.
- A implementation of a-star algorithm
- A heuristic function for a-start that is based on euclidean distance

### Implementing Your Path Planning Algorithm

#### 1. Set your global home position
Here students should read the first line of the csv file, extract lat0 and lon0 as floating point values and use the self.set_home_position() method to set global home. Explain briefly how you accomplished this in your code.

This has been implemented as follows
```
    with open("colliders.csv") as f:
            line = f.readline()
            lon = line.split(",")[0]
            lat = line.split(",")[1]
            lon0 = float(lon.split()[1])
            lat0 = float(lat.split()[1])
```

and then a call to 

And here is a lovely picture of our downtown San Francisco environment from above!
![Map of SF](./misc/map.png)

#### 2. Set your current local position
Here as long as you successfully determine your local position relative to global home you'll be all set. Explain briefly how you accomplished this in your code.

Implemented in line 137

Meanwhile, here's a picture of me flying through the trees!
![Forest Flying](./misc/in_the_trees.png)

#### 3. Set grid start position from local position
This is another step in adding flexibility to the start location. As long as it works you're good to go!

I did the following 3 steps:
current_global = self.global_position
current_local = global_to_local(current_global, self.global_home)
grid_start = (int(current_local[0]-north_offset), int(current_local[1]-east_offset))

#### 4. Set grid goal position from geodetic coords
This step is to add flexibility to the desired goal location. Should be able to choose any (lat, lon) within the map and have it rendered to a goal location on the grid.

free_cells_indices =  np.where(grid == 0)
free_cell_idx = np.random.randint(len(free_cells_indices[0]))
grid_goal = (free_cells_indices[0][free_cell_idx], free_cells_indices[1][free_cell_idx])

#### 5. Modify A* to include diagonal motion (or replace A* altogether)
Minimal requirement here is to modify the code in planning_utils() to update the A* implementation to include diagonal motions on the grid that have a cost of sqrt(2), but more creative solutions are welcome. Explain the code you used to accomplish this step.

I have modified lines 59-62 (adding new enum values with sqrt(2) cost)
and also lines 93-100 to check for boundaries

#### 6. Cull waypoints 
For this step you can use a collinearity test or ray tracing method like Bresenham. The idea is simply to prune your path of unnecessary waypoints. Explain the code you used to accomplish this step.

Add a method in planning_utils called prune_path, the method uses collinearity test for any consecutive 3 points and removes any middle point that falls exactly between the first and third.


### Execute the flight
#### 1. Does it work?
It works!

### Double check that you've met specifications for each of the [rubric](https://review.udacity.com/#!/rubrics/1534/view) points.
  
# Extra Challenges: Real World Planning

For an extra challenge, consider implementing some of the techniques described in the "Real World Planning" lesson. You could try implementing a vehicle model to take dynamic constraints into account, or implement a replanning method to invoke if you get off course or encounter unexpected obstacles.


