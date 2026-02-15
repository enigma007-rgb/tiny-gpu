# How tiny-gpu Works (Simple Explanation)

## The Big Picture

Imagine you need to add two lists of 1000 numbers together. A regular computer (CPU) is like having one very smart worker who adds them one pair at a time: 1+1, then 2+2, then 3+3... taking 1000 steps.

A GPU is like having 1000 decent workers who each handle ONE pair simultaneously. They all finish in just 1 step. That's the core idea: **many simple workers doing the same task on different data at once**.

---

## The Main Parts

### **The Boss (Dispatcher)**
When you give the GPU work to do, the boss organizes workers into **teams** (called "blocks"). Each team gets sent to a workshop to do their job. The boss makes sure every team gets assigned and tracks when everyone's done.

### **The Workshops (Compute Cores)**
These are where teams actually work. Each workshop has:
- **Calculators** (one for each worker) - for doing math
- **Notepads** (registers) - where each worker writes down their personal numbers
- **Messenger** - who fetches instructions and gets data from the warehouse

### **The Warehouse (Memory)**
This is where all the data lives - the numbers you want to add, multiply, etc. The problem? The warehouse is far away, so fetching stuff takes time. This is the GPU's biggest bottleneck.

### **The Cache**
Think of this like a small shelf right next to the workshop. If workers need the same data repeatedly, they keep a copy on the shelf instead of walking to the far-away warehouse every time.

---

## How a Job Gets Done

Let's use the example of adding two lists of 8 numbers:

**List A:** [1, 2, 3, 4, 5, 6, 7, 8]  
**List B:** [1, 2, 3, 4, 5, 6, 7, 8]  
**We want:** [2, 4, 6, 8, 10, 12, 14, 16]

### Step 1: Setup
You load:
- The **recipe** (program instructions) into the instruction warehouse
- The **ingredients** (List A and List B) into the data warehouse
- Tell the boss: "I need 8 workers for this job"

### Step 2: Assignment
The boss creates a team of 8 workers and sends them to a workshop. Each worker gets:
- A **worker number** (Worker #0, Worker #1, etc.)
- The same **recipe to follow** (instructions)
- Their own **notepad** to work with

### Step 3: Following the Recipe
The recipe says:
1. "Figure out which number YOU should handle" (Worker #0 handles position 0, Worker #1 handles position 1, etc.)
2. "Go to the warehouse and get YOUR number from List A"
3. "Go to the warehouse and get YOUR number from List B"
4. "Add them together on your calculator"
5. "Write the answer back to the warehouse"

### Step 4: The Magic Part
ALL 8 workers do these steps **at the same time**. They're all:
- Going to the warehouse at the same time
- Calculating at the same time
- Writing answers at the same time

Worker #0 calculates 1+1=2  
Worker #1 calculates 2+2=4  
Worker #2 calculates 3+3=6  
...and so on, **all simultaneously**.

### Step 5: Done!
When all workers finish, the boss says "Job complete!" and the warehouse now has your answer: [2, 4, 6, 8, 10, 12, 14, 16]

---

## The Clever Bits

### **Same Recipe, Different Data**
The secret is that every worker follows the EXACT same recipe, but their "worker number" makes them work on different data. This is why it's called "Same Instruction, Multiple Data" (SIMD).

Think of it like a dance class: everyone does the same moves at the same time, but each person is in their own spot on the floor.

### **The Waiting Problem**
The warehouse (memory) is slow. When workers request data, they have to WAIT. This is like workers standing around doing nothing while the messenger runs to the warehouse.

Real GPUs are very clever about:
- **Caching**: Keeping frequently-needed stuff on the nearby shelf
- **Batching requests**: Combining multiple workers' requests into one warehouse trip
- **Overlapping work**: While some workers wait for data, other workers do calculations

This toy GPU keeps it simple and just waits.

### **Loops and Decisions**
The recipe can have loops (repeat steps) and decisions (if this, do that). Workers use a **comparison checker** that remembers "is my number bigger, smaller, or equal?" and can jump to different recipe steps based on that.

---

## Why This Matters

**Matrix math** (which powers AI and graphics) involves doing the same operation thousands or millions of times on different numbers. 

- Rendering an image? Do color calculations for millions of pixels (each pixel = one worker)
- Training AI? Multiply huge grids of numbers (each grid position = one worker)

GPUs can have **thousands** of these workers operating simultaneously, making them incredibly fast for these repetitive tasks.

---

## What This Toy Version Teaches

This tiny-gpu is intentionally simple (like a model car engine) so you can actually understand how it works:

- Only 256 spots in memory (real GPUs have billions)
- Simple 11-instruction recipe language
- Assumes all workers stay in sync (real GPUs handle workers going different directions)
- One team at a time (real GPUs juggle many teams)

But the **core principles** are identical to the GPUs in your phone, gaming PC, or AI data centers. It's just scaled way, way up!

-----

# Real Example: Adding Two Lists of Numbers

Let's walk through **exactly** what happens when tiny-gpu adds these two lists:

**List A:** [1, 2, 3, 4, 5, 6, 7, 8]  
**List B:** [1, 2, 3, 4, 5, 6, 7, 8]  
**Result:** [2, 4, 6, 8, 10, 12, 14, 16]

I'll show you every single step, like watching it in slow motion.

---

## BEFORE WE START: Loading Everything

### The Warehouse Shelves (Memory)
Imagine shelves numbered 0 to 255. We place our data:

```
Shelf 0:  1  ← First number of List A
Shelf 1:  2
Shelf 2:  3
Shelf 3:  4
Shelf 4:  5
Shelf 5:  6
Shelf 6:  7
Shelf 7:  8  ← Last number of List A

Shelf 8:  1  ← First number of List B
Shelf 9:  2
Shelf 10: 3
Shelf 11: 4
Shelf 12: 5
Shelf 13: 6
Shelf 14: 7
Shelf 15: 8  ← Last number of List B

Shelf 16-23: [empty] ← This is where answers will go
```

### The Recipe Book (Program Instructions)
We also load the recipe (program) into a separate instruction shelf:

```
Instruction 0: "Multiply your worker-team-number by team-size"
Instruction 1: "Add your personal worker-number to that"
Instruction 2: "Remember: List A starts at shelf 0"
Instruction 3: "Remember: List B starts at shelf 8"
Instruction 4: "Remember: Results go starting at shelf 16"
Instruction 5: "Calculate which shelf has YOUR A number"
Instruction 6: "Go get YOUR A number from that shelf"
Instruction 7: "Calculate which shelf has YOUR B number"
Instruction 8: "Go get YOUR B number from that shelf"
Instruction 9: "Add A + B together"
Instruction 10: "Calculate which shelf YOUR answer goes to"
Instruction 11: "Put your answer on that shelf"
Instruction 12: "Tell the boss you're done"
```

### The Control Panel
We set the dial to: **"8 workers needed"**

---

## STEP 1: THE BOSS CREATES A TEAM

The **Dispatcher** (boss) says:
- "I need a team of 8 workers"
- "This is team #0 (the first and only team)"
- "Team, go to Workshop #1"

The boss creates 8 workers and sends them to Workshop #1:

```
Worker #0 - goes to Workshop #1
Worker #1 - goes to Workshop #1
Worker #2 - goes to Workshop #1
Worker #3 - goes to Workshop #1
Worker #4 - goes to Workshop #1
Worker #5 - goes to Workshop #1
Worker #6 - goes to Workshop #1
Worker #7 - goes to Workshop #1
```

---

## STEP 2: EACH WORKER GETS THEIR SETUP

Inside Workshop #1, each worker gets:

### **Their own notepad (registers):**
```
Worker #0's notepad:
  R0-R12: [blank spaces for working]
  SPECIAL NOTE #1: "Your team number is: 0"
  SPECIAL NOTE #2: "Your team has: 8 workers"
  SPECIAL NOTE #3: "You are worker: 0"

Worker #1's notepad:
  R0-R12: [blank spaces for working]
  SPECIAL NOTE #1: "Your team number is: 0"
  SPECIAL NOTE #2: "Your team has: 8 workers"
  SPECIAL NOTE #3: "You are worker: 1"

Worker #2's notepad:
  ...
  SPECIAL NOTE #3: "You are worker: 2"

[and so on for workers #3, #4, #5, #6, #7]
```

### **Their own calculator (ALU)**
Each worker gets a calculator that can add, subtract, multiply, and divide.

### **Their own position tracker (Program Counter)**
Everyone starts at "Instruction 0"

---

## STEP 3: EVERYONE STARTS FOLLOWING THE RECIPE TOGETHER

### **Instruction 0: "Multiply your team-number by team-size"**

All 8 workers read instruction 0 at the same time:

```
Worker #0: My team# is 0, team-size is 8
           0 × 8 = 0
           Writes "0" on notepad space R0

Worker #1: My team# is 0, team-size is 8
           0 × 8 = 0
           Writes "0" on notepad space R0

Worker #2-7: [same calculation, all get 0]
```

**Why?** We only have one team (team #0), so this equals 0 for everyone. In a bigger job with multiple teams, this would be different.

---

### **Instruction 1: "Add your personal worker-number to that"**

```
Worker #0: R0 was 0, my worker# is 0
           0 + 0 = 0
           Updates R0 to "0"

Worker #1: R0 was 0, my worker# is 1
           0 + 1 = 1
           Updates R0 to "1"

Worker #2: R0 was 0, my worker# is 2
           0 + 2 = 2
           Updates R0 to "2"

Worker #3: Updates R0 to "3"
Worker #4: Updates R0 to "4"
Worker #5: Updates R0 to "5"
Worker #6: Updates R0 to "6"
Worker #7: Updates R0 to "7"
```

**Now each worker has their unique ID number!** This is the magic - everyone did the same calculation but got different results because of their personal worker number.

---

### **Instructions 2-4: "Remember the shelf locations"**

```
Worker #0: 
  Writes "0" in R1  (List A starts at shelf 0)
  Writes "8" in R2  (List B starts at shelf 8)
  Writes "16" in R3 (Results start at shelf 16)

Workers #1-7: [All write the same numbers - 0, 8, 16]
```

Everyone now knows where the data lives.

---

### **Instruction 5: "Calculate which shelf has YOUR A number"**

```
Worker #0: R1=0 (List A base), R0=0 (my ID)
           0 + 0 = 0
           Writes "0" in R4 → "My A number is on shelf 0"

Worker #1: 0 + 1 = 1
           Writes "1" in R4 → "My A number is on shelf 1"

Worker #2: Writes "2" in R4 → shelf 2
Worker #3: Writes "3" in R4 → shelf 3
Worker #4: Writes "4" in R4 → shelf 4
Worker #5: Writes "5" in R4 → shelf 5
Worker #6: Writes "6" in R4 → shelf 6
Worker #7: Writes "7" in R4 → shelf 7
```

Each worker now knows which shelf to visit!

---

### **Instruction 6: "Go get YOUR A number from that shelf"**

This is where things get interesting! All workers need to go to the warehouse at the SAME TIME.

```
Worker #0: Sends messenger to warehouse: "Get me shelf 0"
Worker #1: Sends messenger to warehouse: "Get me shelf 1"
Worker #2: Sends messenger to warehouse: "Get me shelf 2"
Worker #3: Sends messenger to warehouse: "Get me shelf 3"
Worker #4: Sends messenger to warehouse: "Get me shelf 4"
Worker #5: Sends messenger to warehouse: "Get me shelf 5"
Worker #6: Sends messenger to warehouse: "Get me shelf 6"
Worker #7: Sends messenger to warehouse: "Get me shelf 7"
```

**The memory controller receives 8 requests at once!**

The warehouse can only handle a few requests at a time (let's say 2), so the memory controller creates a queue:

```
Processing now: Worker #0's request, Worker #1's request
Waiting: Workers #2, #3, #4, #5, #6, #7
```

**Time passes... (this is the slow part!)**

After a few clock cycles, all requests are processed:

```
Worker #0 receives: 1 (from shelf 0) → updates R4 to "1"
Worker #1 receives: 2 (from shelf 1) → updates R4 to "2"
Worker #2 receives: 3 → updates R4 to "3"
Worker #3 receives: 4 → updates R4 to "4"
Worker #4 receives: 5 → updates R4 to "5"
Worker #5 receives: 6 → updates R4 to "6"
Worker #6 receives: 7 → updates R4 to "7"
Worker #7 receives: 8 → updates R4 to "8"
```

Now everyone has their A number!

---

### **Instruction 7-8: Get the B numbers (same process)**

```
Worker #0: Calculates shelf 8, gets value 1 → R5 = "1"
Worker #1: Calculates shelf 9, gets value 2 → R5 = "2"
Worker #2: Calculates shelf 10, gets value 3 → R5 = "3"
Worker #3: Calculates shelf 11, gets value 4 → R5 = "4"
Worker #4: Calculates shelf 12, gets value 5 → R5 = "5"
Worker #5: Calculates shelf 13, gets value 6 → R5 = "6"
Worker #6: Calculates shelf 14, gets value 7 → R5 = "7"
Worker #7: Calculates shelf 15, gets value 8 → R5 = "8"
```

---

### **Instruction 9: "Add A + B together"**

NOW THE MAGIC HAPPENS! All calculators activate simultaneously:

```
Worker #0: R4=1, R5=1  →  1+1=2  →  R6="2"
Worker #1: R4=2, R5=2  →  2+2=4  →  R6="4"
Worker #2: R4=3, R5=3  →  3+3=6  →  R6="6"
Worker #3: R4=4, R5=4  →  4+4=8  →  R6="8"
Worker #4: R4=5, R5=5  →  5+5=10  →  R6="10"
Worker #5: R4=6, R5=6  →  6+6=12  →  R6="12"
Worker #6: R4=7, R5=7  →  7+7=14  →  R6="14"
Worker #7: R4=8, R5=8  →  8+8=16  →  R6="16"
```

**All 8 additions happen in ONE clock cycle!** This is the GPU's superpower.

---

### **Instruction 10: "Calculate which shelf YOUR answer goes to"**

```
Worker #0: R3=16 (result base), R0=0 (my ID)
           16 + 0 = 16 → R7="16" (my answer goes to shelf 16)

Worker #1: 16 + 1 = 17 → R7="17"
Worker #2: 16 + 2 = 18 → R7="18"
Worker #3: 16 + 3 = 19 → R7="19"
Worker #4: 16 + 4 = 20 → R7="20"
Worker #5: 16 + 5 = 21 → R7="21"
Worker #6: 16 + 6 = 22 → R7="22"
Worker #7: 16 + 7 = 23 → R7="23"
```

---

### **Instruction 11: "Put your answer on that shelf"**

All workers send their answers to the warehouse:

```
Worker #0: "Store my answer (2) on shelf 16"
Worker #1: "Store my answer (4) on shelf 17"
Worker #2: "Store my answer (6) on shelf 18"
Worker #3: "Store my answer (8) on shelf 19"
Worker #4: "Store my answer (10) on shelf 20"
Worker #5: "Store my answer (12) on shelf 21"
Worker #6: "Store my answer (14) on shelf 22"
Worker #7: "Store my answer (16) on shelf 23"
```

The memory controller queues these writes and processes them.

**After a few cycles, the warehouse shelves now contain:**

```
Shelf 16: 2
Shelf 17: 4
Shelf 18: 6
Shelf 19: 8
Shelf 20: 10
Shelf 21: 12
Shelf 22: 14
Shelf 23: 16
```

---

### **Instruction 12: "Tell the boss you're done"**

```
All workers: "I'm finished!"
```

The workshop coordinator tells the dispatcher: "Team finished!"

The dispatcher looks at its list: "No more teams to process. Job complete!"

---

## THE FINAL RESULT

The warehouse memory now looks like:

```
Shelf 0-7:   [1,2,3,4,5,6,7,8]   ← Original List A
Shelf 8-15:  [1,2,3,4,5,6,7,8]   ← Original List B
Shelf 16-23: [2,4,6,8,10,12,14,16] ← OUR ANSWER! ✓
```

---

## THE KEY INSIGHT

Notice what happened:
- **Same instructions** for everyone (all workers followed the same recipe)
- **Different data** for each worker (because of their unique worker number)
- **Parallel execution** - all 8 additions happened simultaneously

If this was a CPU, it would do:
1. Get A[0] and B[0], add them, store result
2. Get A[1] and B[1], add them, store result
3. Get A[2] and B[2], add them, store result
... (8 separate steps)

The GPU does all 8 in parallel! With a real GPU having thousands of workers, this advantage becomes MASSIVE for tasks like AI training or graphics rendering.
