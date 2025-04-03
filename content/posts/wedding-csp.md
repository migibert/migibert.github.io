+++
title = 'Optimizing Wedding Seating with Constraint Programming'
description = 'Planning a wedding seating chart can be stressful, balancing family ties, friendships, and conflicts. This post explores how Constraint Programming (CP) and Google OR-Tools can optimize wedding seating arrangements by maximizing affinities and minimizing conflicts. Learn how computational problem-solving can take the guesswork out of event planning!'
date = 2025-03-28
draft = false
categories = ['theory of constraints', 'software engineering']
tags = ['csp']
+++

## Introduction

Planning a wedding involves many logistical challenges, but one of the most delicate tasks is seating arrangements. A wedding is a mix of family members, lifelong friends, and work colleagues, each with their own histories, relationships, and preferences. Some guests want to sit together, others should be kept apart, and family members often have priority seating near the married couple.

A manually crafted seating chart can lead to compromises, overlooked conflicts, and unnecessary stress. Instead, we can turn to **Constraint Programming (CP)**, a powerful approach that finds an optimal arrangement based on defined rules and preferences.

## Defining the Constraints

A well-designed seating arrangement should balance social dynamics while respecting practical limitations. [This repository](https://github.com/migibert/wedding-csp) shows a possible implementation.

### **Family Priorities**
   - The **married couple** should sit together (strongest affinity, 100).
   - **Parents** should sit close to their child (50 affinity) and their child-in-law (25 affinity).
   - **Immediate family members** (spouses, children) have a moderate affinity (10).

### **Friendships and Social Preferences**
   - Friends with strong bonds have a preference to sit together (affinity values between 5 and 10).
   - Work colleagues may prefer to sit together but not as strongly as family.

### **Avoiding Conflicts**
   - Some guests have personal history or past disagreements, meaning they should be seated apart.
   - We assign **negative affinity values** to these relationships, ensuring the solver tries to avoid placing them at the same table.

### **Table Size Limits**
   - Each table has a fixed number of seats, meaning the solver must distribute guests while respecting capacity constraints.

## How Constraint Programming Helps

Constraint Programming (CP) is a technique used to solve combinatorial problems where multiple constraints interact. Instead of trying every possible combination manually, we define rules, and the solver finds the best arrangement automatically.

We use **Google OR-Tools**, a powerful open-source solver, to:
1. **Define Variables**: Each guest is assigned a table.
2. **Apply Constraints**: The solver ensures that each guest is at one table, that table capacities are not exceeded, and that affinity relationships are respected.
3. **Optimize the Seating**: The solver maximizes positive affinities while minimizing conflicts.

### Why Use OR-Tools?
OR-Tools is designed for complex decision-making problems like scheduling, resource allocation, and, in our case, wedding seating. Unlike brute-force approaches, which are computationally expensive, OR-Tools **prunes impossible solutions early** and finds near-optimal placements efficiently.

### What does it look like ?

```
from ortools.sat.python import cp_model

def solve_seating(guests, affinities, table_capacities):
    model = cp_model.CpModel()
    num_guests = len(guests)
    num_tables = len(table_capacities)

    # Create variables: guest assignments to tables
    table_vars = [model.NewIntVar(0, num_tables - 1, f'guest_{i}') for i in range(num_guests)]

    # Table capacity constraints
    for table in range(num_tables):
        model.Add(sum(table_vars[i] == table for i in range(num_guests)) <= table_capacities[table])

    # Affinity constraints: maximize overall affinity
    affinity_sum = sum(
        affinities[i][j] * (table_vars[i] == table_vars[j])
        for i in range(num_guests) for j in range(i + 1, num_guests)
    )
    model.Maximize(affinity_sum)

    # Solve model
    solver = cp_model.CpSolver()
    status = solver.Solve(model)

    if status == cp_model.OPTIMAL or status == cp_model.FEASIBLE:
        seating = {guests[i]: f'Table {solver.Value(table_vars[i])}' for i in range(num_guests)}
        return seating
    return "No feasible seating arrangement found."
```


## Conclusion
By using Constraint Programming, we can solve the wedding seating problem in a structured and optimized way. This approach eliminates the guesswork, ensures all constraints are considered, and produces a well-balanced seating arrangement that respects family ties, friendships, and social dynamics.

With this method, wedding planners (or engaged couples!) can focus on celebrating rather than stressing over seating charts.

