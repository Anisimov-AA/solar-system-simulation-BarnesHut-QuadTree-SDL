# Solar System Simulation with Barnes-Hut Quad-Tree Algorithm

A real-time N-body gravitational simulation of the solar system with 9 planets and 200 asteroids using efficient Barnes-Hut quad-tree spatial optimization.

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
The total number of calculations per frame is roughly n², which is 209 * 209 = 43,681  
  
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

## 📋 Key Features

This simulation demonstrates advanced computational physics with:

- Barnes-Hut Quad-Tree Algorithm: O(N log N) gravitational calculations using spatial quad-trees
- Solar System: 9 planets with realistic masses and orbital distances
- Dynamic Asteroid Belt: 200 asteroids between Mars and Jupiter
- Real-time Visualization: Interactive SDL2 rendering with planetary trajectory trails
- Interactive Controls: Zoom (40x-400x) and variable speed (0.0001-0.1 time-step)
- Data Logging: Automatic CSV export of simulation data

## 🎮 Controls

- Zoom: +/- buttons (top-right corner)
- Speed: +/- buttons (middle-right corner)
- Exit: Close window

## 🏗️ Architecture

The simulation uses a sophisticated quad-tree based approach:

```
main.c  
├── Data Structures  
│   ├── CelestialBody                    # Universal body structure for quad-tree  
│   ├── Planet                           # Solar system planets with orbital data  
│   └── QuadTreeNode                     # Spatial partitioning for efficient calculations  
├── Barnes-Hut Implementation  
│   ├── create_quadtree()                # Spatial tree construction  
│   ├── insert_body()                    # Dynamic body insertion  
│   ├── calculate_center_of_mass()       # Mass distribution calculation  
│   └── calculate_force_from_quadtree()  # O(N log N) force computation  
├── Physics Engine  
│   ├── update_body()                    # Numerical integration (Euler method)  
│   └── update_simulation_barnes_hut()   # Main simulation loop  
└── Rendering System  
    ├── render_bodies()                  # Planet and asteroid visualization  
    ├── DrawButton()                     # Interactive UI elements  
    └── trajectory rendering             # Orbital path display  
```

## 🛠️ Technologies

- C - High-performance systems programming
- SDL2 - Hardware-accelerated graphics and input
- SDL2_ttf - Font rendering for UI elements
- GCC - GNU Compiler Collection

## ⚙️ Installation & Setup

**Prerequisites**  
First, install the SDL2 development library:  

Ubuntu/Debian:  

```bash
sudo apt update  
sudo apt install libsdl2-dev  
```

macOS (using Homebrew):
  
```bash
brew install sdl2
```

**Compilation**  
Compile using the provided Makefile:  

```bash
make
```

**Running the Simulation**  
Execute the compiled binary:  

```bash
./solar_system
```
