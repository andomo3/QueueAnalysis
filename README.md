Here’s a README-ready Markdown version of the combined report you can drop straight into your repo:

---

# Final Report

**Author:** Abba Otieno Ndomo

## Data Collection Methodology

### Objectives and Scope

We collected vehicle and pedestrian data at a T-intersection with three vehicle approaches (Ferst Dr K, Hemphill H, Ferst Dr C) and associated crosswalks. Our goal was to record arrival, queue progression, and exit times so that we could construct inter-arrival, service, and waiting-time distributions for later queueing analysis.

### Team Roles

For each vehicle approach, a road observation team included:

* **Arrival Recorder**: Noted when a vehicle joined the back of the queue.
* **Exit Recorder**: Noted when a vehicle cleared the intersection.
* **Data Logger**: Entered Vehicle ID, Vehicle Type, Arrival Time, Stop Sign Arrival (if used), Exit Time, and Direction From/To into a shared spreadsheet.

A dedicated **Pedestrian Observer** recorded pedestrian arrival at the crosswalk, crossing start time, and exit time.

### Data Schema and Tools

* **Vehicle sheets** contained:

  * Vehicle ID, Vehicle Type, Arrival Time, Stop Sign Arrival, Exit Time, Direction From/To
  * Automatically computed processing times
  * Counts of cars still in queue (when Exit Time was blank)

* **Pedestrian sheets** contained:

  * Pedestrian ID, Arrival Time, Crossing Start, Exit Time
  * Automatically computed waiting and crossing durations

* A **summary sheet** aggregated:

  * Cars in queue per road and in total
  * Pedestrians waiting, crossing, and cumulative counts

Observers used timestamp shortcuts (e.g., `Ctrl + Shift + ;`) and consistent direction and vehicle-type codes to reduce typing errors.

### Procedures and Quality Checks

* Times were logged **immediately** at each event (arrival, stop, exit) to avoid recall errors.
* Periodic cross-checks between Arrival and Exit recorders and the Data Logger helped reconcile missing or inconsistent rows.
* The final dataset consisted of:

  * Per-approach vehicle records
  * Pedestrian crossing records
  * Summary totals for sanity checks before downstream analysis

---

## Queueing Analysis of Intersection with Pedestrians

We analysed vehicle data from a signalised intersection with three approach streams and observable pedestrian crossings. For each car, we parsed arrival, stop, and exit times. **Inter-arrival time** was defined as the difference between consecutive arrivals, **service time** as the time-in-the-intersection (exit minus stop), and **waiting time** as stop minus arrival. The objective was to model the system as a single queue with a single server using QPLEX, then compare predicted waiting times to observed waits in order to assess the impact of pedestrians on vehicle delay.

### Data Cleaning and Empirical Distributions

![Snapshot of helper methods for cleaning the dataset](cleaning.png)

We cleaned the data by replacing any non-positive inter-arrival, entry, service, or waiting times with the minimum positive value observed in that series and removing missing values. We then formed empirical PMFs by binning times into **5-second intervals**, capping support at **30 bins (0–150s)** to keep the QPLEX state space tractable.

These PMFs were supplied to the `StandardMultiserver` model in QPLEX with `number_of_servers = 1`. After a 200-step burn-in, we sampled over 400 steps, truncating the state space at 400 states.

![Snapshot of the cleaned dataset](image.png)

### Results and Interpretation

For the combined vehicle stream, the mean inter-arrival time was approximately
**E[A] ≈ 7.3 s**, and the mean service time **E[S] ≈ 15.0 s**, giving:

* Arrival rate: **λ ≈ 0.137 cars/s**
* Traffic intensity: **ρ = λ · E[S] ≈ 2.05 > 1**

QPLEX returned a truncated steady-state distribution with:

* **E[N] ≈ 4066** (mean number in system)
* **E[Q] ≈ 4065** (mean queue length)

Using Little’s Law, the implied average waiting time was:

* **Wₛ (queue) ≈ 29,700 s (~8.25 hours)**

These values are **not physically realistic**, since observed waits at the intersection are on the order of **tens of seconds**. Instead, they indicate that, **under the assignment’s one-queue/one-server framing, the system is theoretically unstable**: pedestrian-inflated service times combined with the observed arrival process exceed the capacity of a single server.

Thus, QPLEX **qualitatively** signals that pedestrians **significantly reduce effective capacity**, but **realistic quantitative predictions** would require:

* Multiple servers (e.g., parallel lanes or phases), and/or
* Reduced service times (e.g., a no-pedestrian scenario), and/or
* Lower effective arrival rates due to upstream feedback and congestion effects.
