Real-time N-body gravitational simulation with **Barnes-Hut quad-tree optimization** - simulates 209 bodies (9 planets + 200 asteroids) efficiently

<div align="center">
  <table>
    <tr>
      <td style="text-align: center; padding: 10px;">
        <img src="img/solar_system.jpg" alt="Solar System Overview" height="400"/>
        <br><b>Solar System</b>
      </td>
      <td style="text-align: center; padding: 10px;">
        <img src="img/barnes-hut.jpg" alt="Barnes-Hut Visualization" height="400"/>
        <br><b>Barnes-Hut</b>
      </td>
    </tr>
  </table>
</div>

##  Problem Underastanding
Objects (n): 9 planets + 200 asteroids = 209 total objects  
The Goal: For every frame of the simulation, you need to calculate the gravitational forces and check for potential collisions for every object  

**The O(n^2) Brute-Force Method**  
This method is simple: you create a checklist of every possible pair of objects and run the calculations.  
  
Here's how it would work:  
1. Pick Earth. You calculate its interaction with all 208 other objects (8 planets and 200 asteroids)
2. Pick Mars. You calculate its interaction with all 208 other objects
3. Pick Asteroid #1. You calculate its interaction with all 208 other objects
4. ...and so on, until you've done this for every planet and every asteroid
  
The Math:  
The total number of calculations per frame is roughly n^2, which is 209 * 209 = 43,681  
  
So, to render a single frame of your simulation, your computer has to perform over 43,000 calculations. This is incredibly inefficient  

**The O(n log n) QuadTree Method**  
This method is clever. It recognizes that you don't need to check objects that are on the other side of the solar system from each other  
  
Here's how it works:  

1. Build the QuadTree (Organize the Space):
  - First, your program draws a giant box around the entire solar system and divides it into four large quadrants
  - It looks at the top-right quadrant. Let's say it contains Jupiter, Saturn, and 80 asteroids. Since that's still a lot of objects, it divides that quadrant into four smaller sub-quadrants
  - It keeps dividing any box that is too crowded until every box contains only a handful of objects. The inner solar system, with many planets close together, will be divided into many small boxes

2. Calculate Interactions Intelligently:
Now, let's calculate the forces on Earth.
  - Instead of checking all 208 other objects, the program first finds the small box that Earth is in. This box might also contain Venus, Mars, and 3 asteroids.
  - For close objects: It performs precise, one-on-one calculations with these 5 nearby objects.
  - For distant objects: The program looks at a very distant box (e.g., the one containing Saturn and its 20 asteroids). Instead of doing 21 separate calculations, the Barnes-Hut algorithm allows it to bundle that entire group together. It calculates their combined center of mass and treats them as one big object. So, it performs just one approximate calculation for that entire distant group.

The Math:  
The number of calculations is now roughly n * log n  
- n = 209
- log2(209) is about 8 (since 2^8 = 256)
- Total calculations per frame = 209 * 8 ≈ 1,700

By using a QuadTree, we made our simulation more than 25 times faster. We achieved this by smartly organizing our data so we could ignore calculations that didn't matter, which is the core of efficient algorithm design

## Setup

**Requirements:** C compiler (GCC), SDL2, SDL2_ttf

1. install dependencies (Ubuntu)
```bash
sudo apt update && sudo apt install libsdl2-dev libsdl2-ttf-dev
```

2. compile
```bash
make
```

3. run
```bash
./solar_system
```

## Usage

**Controls:**
- **Zoom:** +/− buttons (top-right) - scale from 40x to 400x
- **Speed:** +/− buttons (middle-right) - time-step from 0.0001 to 0.1
- **Exit:** Close window

**Simulation:**
- 9 planets with realistic masses and orbital distances
- 200 asteroids in belt between Mars and Jupiter
- Planetary trajectory trails show orbital paths
- CSV data automatically exported to file

## Architecture
```
main.c
├── Data Structures
│   ├── CelestialBody   # position, velocity, mass, force
│   ├── Planet          # solar system data (orbital parameters)
│   └── QuadTreeNode    # spatial partition with center of mass
├── Barnes-Hut Algorithm
│   ├── create_quadtree()                # build spatial tree
│   ├── insert_body()                    # add body to correct quadrant
│   ├── calculate_center_of_mass()       # compute node mass distribution
│   └── calculate_force_from_quadtree()  # O(n log n) force calculation
├── Physics
│   ├── update_body()                    # Euler integration (F=ma)
│   └── update_simulation_barnes_hut()   # main simulation loop
└── Rendering
    ├── render_bodies()       # draw planets/asteroids
    ├── DrawButton()          # UI controls
    └── trajectory_rendering  # orbital path trails
```
