import gurobipy as gp
from gurobipy import GRB

# Number of stops (example size)
n_stops = 5

# Boardings and alightings at each stop
b = [10, 20, 15, 5, 0]  # example boarding counts
a = [5, 10, 10, 10, 15]  # example alighting counts

# Create a new model
model = gp.Model("OD_Matrix_Optimization")

# Create variables for x (flows) and y (auxiliary variables for absolute values)
x = model.addVars(n_stops, n_stops, lb=0, name="x")
y = model.addVars(n_stops, n_stops, lb=0, name="y")

# Objective: Minimize the sum of y_ij
model.setObjective(gp.quicksum(y[i,j] for i in range(n_stops) for j in range(n_stops)), GRB.MINIMIZE)

# Flow conservation constraints
for i in range(n_stops):
    model.addConstr(gp.quicksum(x[i,j] for j in range(n_stops)) - gp.quicksum(x[j,i] for j in range(n_stops)) == b[i] - a[i], f"flow_conservation_{i}")

# Trip movement flow constraints (y_ij >= |x_ij|)
for i in range(n_stops):
    for j in range(n_stops):
        model.addConstr(x[i,j] <= y[i,j], f"abs_pos_{i}_{j}")
        model.addConstr(-x[i,j] <= y[i,j], f"abs_neg_{i}_{j}")

# Flow non-negativity constraints are automatically handled by lb=0 in x

# Optimize the model
model.optimize()

# Output the solution
if model.status == GRB.OPTIMAL:
    print("Optimal solution found:")
    for i in range(n_stops):
        for j in range(n_stops):
            print(f"x[{i},{j}] = {x[i,j].x}")
else:
    print("No optimal solution found.")
