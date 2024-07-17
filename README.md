# **Travelling Salesman Problem**

The Traveling Salesman Problem (TSP) is a classic problem in operations research and computer science, focusing on optimization. The problem involves a salesman who must visit a list of cities and return to the starting point. The objective is to find the shortest possible route that visits each city exactly once and returns to the original city, minimizing the total travel distance or cost.
This project utilizes the classic Travelling Salesman Problem (TSP) to optimize routes for concert tours, aiming to find the most efficient path through various cities. The goal is to reduce travel time and costs, enabling artists and tour managers to plan more effective and economical tours.

## Project Overview

The Concert Tour Route Optimizer is designed to solve the logistical challenges of planning concert tours by applying optimization algorithms to determine the shortest possible route that visits each venue once and returns to the origin city. This solution not only saves on travel expenses but also reduces the environmental impact of touring.

## Features

- **Optimization Algorithm**: Implements several TSP solvers to find the best route.
- **Interactive Visualizations**: Offers visual maps of proposed tour routes for easy understanding and decision-making.

## Data Description

The "Concert_Tour_Dataset.xlsx" contains 90 entries structured into three columns: "From," "To," and "Cost (USD)." The "From" and "To" columns list the starting and destination cities for each tour leg, both as text fields. The "Cost (USD)" column details the travel costs between these cities, recorded as a numeric field. This dataset aids in planning efficient concert tour routes by providing essential cost information.

#### Installing libraries and Data Loading
```{python}
import pandas as pd
import cvxpy as cp
import networkx as nx
import matplotlib.pyplot as plt

data_path = "C:/Users/rajpu/Downloads/concert_tour_optimization_dataset.xlsx"
travel_costs_df = pd.read_excel(data_path, sheet_name='Travel Costs')
revenues_df = pd.read_excel(data_path, sheet_name='Revenues')

```
##### Retrieve unique cities from the revenue DataFrame and generate a revenue dictionary and a cost matrix
```{python}
cities = revenues_df['City'].unique()
revenue_dict = revenues_df.set_index('City')['Revenue (USD)'].to_dict()
cost_matrix = pd.pivot_table(travel_costs_df, values='Cost (USD)', index='From', columns='To').fillna(0)
cost_matrix = cost_matrix.reindex(index=cities, columns=cities, fill_value=0)
```

##### x[city]: Binary variable, 1 if city is visited, 0 otherwise, y[from_city, to_city]: Binary variable, 1 if traveling from 'from_city' to 'to_city', 0 otherwise.
```{python}
x = {city: cp.Variable(boolean=True) for city in cities}
y = {(from_city, to_city): cp.Variable(boolean=True) for from_city in cities for to_city in cities}
```

#### Maximize the total revenue from visiting cities minus a penalty for the travel costs between cities.
```{python}
objective = cp.Maximize(sum(revenue_dict[city] * x[city] for city in cities) -
                        0.01 * sum(cost_matrix.at[from_city, to_city] * y[from_city, to_city] for from_city in cities for to_city in cities))
```

#### Ensure that if a city is visited, there must be exactly one departure to another city and exactly one arrival from another city.
```{python}
constraints = [
    sum(y[city, other] for other in cities if other != city) == x[city] for city in cities
]
constraints += [
    sum(y[other, city] for other in cities if other != city) == x[city] for city in cities
]
```
#### Solve the initial problem
```{python}
problem = cp.Problem(objective, constraints)
problem.solve()
```

#### Check for subtours and add constraints if necessary
```{python}
y_vals = {(from_city, to_city): y[from_city, to_city].value for from_city in cities for to_city in cities}
subtours = find_subtours(y_vals)
while subtours:
    for subtour in subtours:
        constraints += [
            sum(y[subtour[i], subtour[(i + 1) % len(subtour)]] for i in range(len(subtour))) <= len(subtour) - 1
        ]
    problem = cp.Problem(objective, constraints)
    problem.solve()
    y_vals = {(from_city, to_city): y[from_city, to_city].value for from_city in cities for to_city in cities}
    subtours = find_subtours(y_vals)
```

#### Extract tour
```{python}
tour_sequence = [(i, j) for (i, j), var in y_vals.items() if var > 0.9]
print("Tour sequence:", tour_sequence)
print("Maximum Revenue: $",problem.value)
```
# **Optimal Path**

![image](https://github.com/user-attachments/assets/137343bc-afcb-4530-9464-4b1593fd6360)

