Load Balancer Simulation — JSQ, JBT, POD Policies

This repository implements and compares different load-balancing policies using a large-scale discrete-event simulation.
The simulator models tasks arriving over time, servers processing queues, and several scheduling strategies commonly used in distributed systems.

Overview

The project simulates the following:

Task arrivals following a hybrid deterministic / exponential distribution

Randomized service times based on a modified Weibull-like model

20 parallel servers, each with its own queue

3 load-balancing policies:

JSQ (Join-the-Shortest-Queue)

JBT (Join-Below-Threshold) — includes custom messaging logic

POD (Power-of-d Choices)

The simulator collects performance metrics such as:

Mean response time

Message overhead (for JBT)

Queue lengths & threshold updates

Server selection decisions

Load Balancing Policies
1. JSQ — Join-the-Shortest-Queue

Each incoming task is routed to the server with:

argmin_i { queue_length(i) + processing_flag(i) }


This minimizes waiting time but requires full system state knowledge.

2. JBT — Join-Below-Threshold

A task is routed using:

Maintain a binary indicator vector

is_server_bt[i] = 1  if  queue_length(i) <= threshold


If one or more servers are below threshold, pick one uniformly at random

Otherwise choose a random server among all

The threshold is periodically updated every time_interval_update time units as:

threshold = min( queue_lengths of d random servers )


Message overhead is tracked as:

msg_cont += 1  whenever a server’s below-threshold flag changes

3. POD — Power-of-d Choices

For each incoming task:

Randomly sample d servers

Assign to the server with the smallest queue among them

Mathematically:

server = argmin { queue_length(s) | s ∈ sample(d) }


This approximates JSQ while requiring very limited information.

Model Components
Inter-arrival times

Hybrid distribution:

With probability $q$, inter-arrival time is constant $T_0$

With probability $(1-q)$, it follows:

T = T0 - mean_Y * log(U)
U ~ Uniform(0,1)

Service time distribution

A modified distribution similar to a bounded Weibull:

X = round( log(r)^2 * beta )
service_time = max(1, min(X, 100 * 2*beta ))


High variance ensures load-balancing behavior is visible under heavy load.

Simulation Output

For each simulation batch:

Mean response time:

response_time = quitting_time - entrance_time


Normalized messaging cost:

msg_ratio = msg_cont / number_of_tasks


Files produced:

dilara_sim_last.csv
jsq.csv
other intermediate logs…


Plots illustrate response time vs. system load:

rho = lambda / mu
rho = [40,…,49.5] / 50

Example Plot (from final code section)

The repository generates a figure:

plt.plot(rho, temp1.mean())
plt.scatter(rho, temp1.mean())


This shows how response time grows as utilization approaches 1 (heavy traffic).

Running the Simulation

Install dependencies:

pip install numpy pandas matplotlib


Run inside Jupyter Notebook or convert to script.
