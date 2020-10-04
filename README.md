
# Amazon delivery trucks
Maintaining a minimal fleet

## Scenario
Amazon Prime Now has 17-hour long days during which it deploys its fleet of trucks. The Order Forecaster for a region is really good at what they do and can give the exact number of trucks needed during each hour block for the coming day. The forecast schedule is in the form of a 17 element array. Our job is to deploy trucks and coordinate with maintenance to decommission trucks that are not needed for the rest of the day. Once a truck is decommisioned, it won't be available until the next day starts. Given the forecaster's schedule, we need to keep a minimal number of trucks in service to fulfill the rest of the day's deliveries, but send back (subtract) any trucks to maintainence as soon as we can. We'll come up with our own scheduling telling them how many trucks to decommission at the start of each hour. We can only deploy (add) trucks at the very beginning of the day, and any leftover trucks at the end of the day are automatically sent to maintenance. For example if the required number of trucks is forecasted as:

| 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12 | 13 | 14 | 15 | 16 |
|---|---|---|---|---|---|---|---|---|---|----|----|----|----|----|----|----|
| 5 | 5 | 5 | 5 | 5 | 5 | 5 | 3 | 4 | 4 | 3  | 1  | 0  | 0  | 0  | 0  | 1  |

The optimal decommisioning schedule is:

| 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7   | 8 | 9 | 10  | 11  | 12 | 13 | 14 | 15 | 16 |
|---|---|---|---|---|---|---|-----|---|---|-----|-----|----|----|----|----|----|
| 5 | 0 | 0 | 0 | 0 | 0 | 0 | \-1 | 0 | 0 | \-1 | \-1 | 0  | 0  | 0  | 0  | 0  |

We start off by adding 5 trucks into delivery. Observe that while we need only 3 trucks during the 7th hour,  we'll need 4 in the 8th hour. So we can remove only 1 truck at the 7th hour to keep at least 4 trucks in service for hours 8 and 9. For the 10th hour, we can decommision 1 truck, leaving 3 to make deliveries. In the next hour we remove 1 truck, leaving the final truck required to make rest of the day's deliveries.

## Approach
We have many different regions to manage and can really give each forecast only one pass through to come up with the precise corresponding deployment/decommisioning schedule. We can do this by reading forecast backwards: at very end of the day where demand is 0 (all deliveries are done), then iterating from the last hour of the day and reading each previous hours one by one to the beginning of the day. Keeping track of the highest demand so far, look at each hour, and subtract the current demand from the highest, if we get a negative value, then that is the number to decomission in the hour+1, if highest minus current hour's demand is positive, then we don't decommission any trucks because we know they'll still be in use in the future. Update the highest demand by taking the max of (current hour's demand vs. highest demand). Then continue through the iteration. The very first hour is finally set to the highest demand.

## Analysis
The time to do this scanning approach is linearly proportional (O(n)) to the number of hours in a day (e.g. 17). An alternative that will take O(n^2) is iterate over the forecast list starting at the beginning, then at each hour, iterate over the rest of the list again to find the max and deciding whether to keep the fleet count same or if it's ok to decomission as much as needed to maintain the current max.


```python
import random
def forecast():
    for _ in range(17):
        yield random.randint(5,20)
```


```python
random.seed(6449)
print(list(forecast()))
```

    [19, 20, 17, 14, 6, 6, 13, 7, 20, 8, 17, 7, 10, 15, 11, 10, 5]
    


```python
random.seed(6449)
most = -1

# Get the entire days demand's
demands = list(forecast())
print(f"Entire day's demands:\n{demands}")

# We'll build the decomissioning list from the day's end and adding each earlier hour back to the beginning of the day
decomissions = []
i = -1
end = -len(demands)
most = max(demands[i], most)
while (i > end):
    i -= 1
    decomissions.insert(0, min(most - demands[i], 0))
    most = max(demands[i], most)

# Finally, add the number of trucks we'll start the day off with
decomissions.insert(0, most)
print(f"Service list:\n{decomissions}")
```

    Entire day's demands:
    [19, 20, 17, 14, 6, 6, 13, 7, 20, 8, 17, 7, 10, 15, 11, 10, 5]
    Service list:
    [20, 0, 0, 0, 0, 0, 0, 0, 0, -3, 0, -2, 0, 0, -4, -1, -5]
    

### What ifs...

- Forecast lists were variable?

- We had 24 hour rolling forecasts, updated with each new hour?

- Maintenance speed improves and we can put trucks back into service after 2 hours?
