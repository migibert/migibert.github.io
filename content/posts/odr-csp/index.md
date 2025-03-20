+++
title = 'From ODR to CSP: Optimizing Decision-Making with Timefold'
description = 'Learn how to optimize decision-making by transforming an Organization Decision Record (ODR) into a Constraint Satisfaction Problem (CSP) using Timefold. This guide walks through modeling constraints, implementing a solver, and automating complex team assignments with AI-driven optimization.'
date = 2025-02-14
draft = false
categories = ['theory of constraints', 'management']
tags = ['csp', 'software engineering', 'decision-making']
+++

## Introduction

The Organization Decision Record (ODR) framework provides a structured approach to decision-making by clearly defining context, constraints, and requirements. However, as decision complexity increases, manually evaluating all possible options becomes impractical. Constraint Satisfaction Problems (CSP) offer a way to systematically optimize decision-making by leveraging constraint-solving algorithms.

In this article, we'll explore how to convert an ODR decision into a CSP problem and implement it using **Timefold**, an open-source optimization engine.

## Understanding the Decision-Making Problem

Let's take the example from the original ODR article:

### Context
- Team A lacks a Product Owner (PO).
- Team B has a PO.
- Hiring externally takes time.

### Constraints
- Each team must have a PO.
- No PO can be assigned to more than one team.
- Hiring externally has a higher cost.

### Considered Options
1. Fire the whole team (unrealistic but considered).
2. Hire an external PO.
   - Open a backfill position.
   - Hire a contractor.
3. Transfer an internal PO.
   - Transfer John Doe from Team A.
   - Transfer Jane Doe from Team B.

Rather than manually scoring these options, we can formulate this as a **Constraint Satisfaction Problem (CSP)** and solve it with Timefold.

## Defining the CSP Model

### Step 1: Define Decision Variables
The decision variables represent the possible assignments of a PO to Team A:
- `poAssignment` can take values `{External, John Doe, Jane Doe, None}`.

### Step 2: Define Constraints
Using Timefold's constraint engine, we encode:
- **Each team must have a PO** → A valid solution must assign a PO to Team A.
- **No PO can be assigned twice** → If Jane Doe is assigned to Team A, she cannot remain in Team B.
- **Hiring cost minimization** → Assigning an internal PO is preferred over hiring externally.

### Step 3: Define an Optimization Function
The objective is to minimize hiring cost while ensuring every team has a PO.

## Implementing the CSP Model with Timefold

```java
@PlanningEntity
public class Team {
    private String name;
    @PlanningVariable(valueRangeProviderRefs = "poRange")
    private ProductOwner assignedPO;
}

@PlanningSolution
public class ODRSolution {
    @PlanningEntityCollectionProperty
    private List<Team> teams;
    
    @ValueRangeProvider(id = "poRange")
    public List<ProductOwner> getAvailablePOs() {
        return List.of(new ProductOwner("John Doe"), new ProductOwner("Jane Doe"), new ProductOwner("External", true));
    }
}

public class ODRConstraintProvider implements ConstraintProvider {
    @Override
    public Constraint[] defineConstraints(ConstraintFactory factory) {
        return new Constraint[] {
            factory.forEach(Team.class)
                .filter(team -> team.getAssignedPO() == null)
                .penalize("Each team must have a PO", HardSoftScore.ONE_HARD),
            
            factory.forEachUniquePair(Team.class, Joiners.equal(Team::getAssignedPO))
                .penalize("No PO duplication", HardSoftScore.ONE_HARD),
            
            factory.forEach(Team.class)
                .filter(team -> team.getAssignedPO() != null && team.getAssignedPO().isExternal())
                .penalize("Minimize external hiring cost", HardSoftScore.ONE_SOFT)
        };
    }
}
```

## Running the Solver

```java
SolverFactory<ODRSolution> solverFactory = SolverFactory.create(new SolverConfig()
    .withSolutionClass(ODRSolution.class)
    .withEntityClasses(Team.class)
    .withConstraintProviderClass(ODRConstraintProvider.class)
    .withTerminationSpentLimit(Duration.ofSeconds(10)));

Solver<ODRSolution> solver = solverFactory.buildSolver();
ODRSolution bestSolution = solver.solve(initialSolution);
```

## Conclusion

By formulating ODR decisions as a CSP, we can leverage constraint solvers like Timefold to automate optimal decision-making.

This approach scales efficiently, making it especially useful for complex organizational decisions. If you're dealing with multiple teams, roles, or constraints, CSP-based optimization could be a game-changer!

