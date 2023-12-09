---
layout: post
title: Cambridge Has (at least) Two Parking Spaces Per Household
---

A [previous post](https://evanfields.net/Cambridge-Street-Parking/) estimated Cambridge's non-metered street parking at 31,474 spaces, approximately one per car-owning household. But these spaces constitute a minority of Cambridge's total parking supply. Here I estimate two other big chunks of our total supply:

* **Parking lots: 43,797 spaces**. The City publishes a nice [geojson](https://github.com/cambridgegis/cambridgegis_data/blob/main/Basemap/Parking_Lots/BASEMAP_ParkingLots.geojson) of parking lots, covering 14.4 million square feet. (14 Harvard Yards!) At 330 square feet per space[^1], that's roughly 44 thousand spots.

* **Residential driveways and garages: 27,368 spaces**. Using a similar methodology[^2] to the previous street parking post, I estimate 27,368 parking spaces in residential driveways and garages. These don't include commercial garages.

Adding these to the on street spaces brings my estimated lower bound of Cambridge parking supply to 107,419:


| Location      | Count       |
|:--------------|------------:|
| On Street     | 31,474      |
| Parking Lot   | 43,797      |
| Driveways     | 27,368      |
| Meters        | 3,400       |
| City Garages  | 1,380       |
| **Total**     | **107,419** |

There are about [50,000](https://www.census.gov/quickfacts/fact/table/cambridgecitymassachusetts/PST045222) households in Cambridge, so this works out to at least two spots per household.

This estimated lower bound doesn't include privately run garages, and perhaps other flavors of parking I'm not thinking of. A quick glance at Parkopedia suggests there are ~20k private garage spaces -- and probably more, because Parkopedia probably isn't exhaustive. Since the population of Cambridge is roughly 118,000, this means it's nearly certain that there are more parking spots than people in Cambridge. You hate to see it.

----

[^1]: Counting the spot itself and aisles / ingress / etc. See [Paved Paradise](https://www.goodreads.com/en/book/show/63329951) chapter 5.
[^2]: As before, I sampled random points from the street centerline data and hand-label the [first 100](https://docs.google.com/spreadsheets/d/1RfaNiy9ffQXxVF3iaUqUZ_rKnj8DcR9nGHgFtD1n24c/edit#gid=1999403840) of these points with how many driveways or residential garages face the street at each sampled point. I tried to be as precise as possible; if a sampled point was near a driveway/garage but didn't seem to be directly perpendicular, I didn't count it. I get an average of 0.2 driveways per road point, meaning 164k linear feet of driveway/garage. Assuming an average driveway width of 9 feet gives us 18k driveways, and 1.5 parking spaces per driveway gets us to 27k spots. 1.5 spots per driveway is a guess, but probably...close to right? Some driveways are small and fit a single vehicle, some fit multiple vehicles end-to-end, and some lead to multi car garages. There's also some philosophical ambiguity; is a 12 foot driveway in front of a garage one spot or two?