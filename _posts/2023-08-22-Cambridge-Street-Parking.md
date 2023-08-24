---
layout: post
title: Cambridge has 31,000 [nearly free] street parking spots
---
## Cambridge has a lot of parking
Depressingly many political arguments in Cambridge are about parking.[[^1]] Despite this, it's not at all obvious how much parking we already have. I've been interested in Cambridge's land use and how much is eaten by cars for a long time, at least since [2021](https://twitter.com/evanjfields/status/1474032536422649866), but doing the full census I crave seems hard and expensive. After reading Henry Grabar's [new book on parking](https://www.goodreads.com/en/book/show/63329951) I was motivated to at least pick the low hanging fruit: estimate the prevalence of street parking, which probably constitutes the plurality of Cambridge's parking supply. Cambridge's GIS department (and thus presumably the City itself) does not have any data on street parking spaces, but using their data on road centerlines and some manual data labeling I have estimated the city contains **31,474** non-metered street parking spaces.

Note that these spots probably don't constitute the *majority* of parking in Cambridge; they don't include:
* Metered parking run by the City [(3400 spaces)](https://www.cambridgema.gov/traffic/parking)
* Garages run by the city [(1380 spaces)](https://www.cambridgema.gov/traffic/parking)
* Private residential parking: home garages, driveways, etc.
* Commercial parking: privately run garages, parking for merchants and customers thereof, office building garages, etc.

Nonetheless, street parking is especially interesting because it's omnipresent, carved out of public space, and nearly free: a Cambridge parking permit costs [$25 per year](https://www.cambridgema.gov/iwantto/applyforaparkingpermit)[[^2]], or a whopping 7 cents per day.

So, is 31,474 a large number of spots? I argue yes; here are some ways to think about it.
* **Dollars of subsidy** Market rate reserved parking spots in Cambridge go for around $300 per *month*. If the City could rent its entire street parking supply at that rate, revenues would be $113 million per year, i.e. ~$1000 per resident per year.
    * Even if all [47.4k](https://datausa.io/profile/geo/cambridge-ma/) households have paid for a parking or visitor permit, that's only 1% of the above revenue.
    * A resident parking permit isn't exactly exchangeable with a reserved spot—the former lets you park nearly anywhere in the city but doesn't guarantee a specific spot; the latter is the opposite. Still, both provide for parking a single car in Cambridge and are thus probably of roughly-comparable value.
    * What would happen to parking prices if parking permits were sold on the open market rather than offered at steep discounts to residents? Demand for market parking would sharply increase: residents who could no longer park with subsidized permits, plus out-of-towners who would like to park in Cambridge. But the market supply of parking would also sharply increase. I'm not sure how these effects would net out in practice. But even if the average market rate of all this parking would be a full order of magnitude lower ($30/month) than the current reserved spot price, that still suggests the current parking is offered to residents >90% below market rates.
* **Square footage** These parking spots represent a combined area of ~5.1 million square feet of public land. For comparison, 5 million square feet is:
    * A bit under the area of MIT's main campus between Vassar St and the Charles (6 million sqft)
    * 5 times the area of Harvard yard
    * Roughly the area of Danehy Park, Cambridge Common, Hoyt Park, RiversidePress Park, the Fresh Pond path, Kingsley Park, Donelly Park, JFK Park, Sennott Park, Joan Lorentz Park, Tim Toomey Park, and University Park...*combined*. We have more parking than parks!
    * 3% of the total area of Cambridge, a larger fraction of the usable land area of Cambridge, and a substantially larger (but unknown by me) fraction of the public land in Cambridge.
* **Parking per household** [31.7k](https://datausa.io/profile/geo/cambridge-ma/) households in Cambridge have a car, so the number of nearly-free street parking spots is remarkably close to 1 per car-owning household. Everyone gets cheap parking! (And pays for everybody else's cheap parking)

## Methodology
Though Cambridge doesn't have an inventory of street parking, it publishes a really nice [dataset](https://github.com/cambridgegis/cambridgegis_data/blob/main/Trans/Street_Centerlines/TRANS_Centerlines.geojson) of street centerlines. 100,000 points sampled uniformly from the collected length of the streets looks like so:

![sampled points]({{ site.baseurl }}/images/parking/manysamples.png "100k sampled points on Cambridge street centerlines")

I hand-labeled the first 100 of these randomly sampled points with how many lanes of street parking the sampled street has *at the sampled point*. For example a point like [(42.36406, -71.11066)](https://www.google.com/maps/search/?api=1&query=42.36406178295066%2C-71.1106646544124) has two lanes of street parking, [this point](https://www.google.com/maps/search/?api=1&query=42.383277909648854%2C-71.12229419906434) has only one lane of street parking *at the sampled point* (there are two lanes of street parking, but at the point sampled one side of the street is a driveway, so not a street parking spot), etc. I tried to be as accurate as possible about the precise points sampled by using a combination of Google maps satellite view and street view. Even if a sampled street had a parking lane on one or both sides, I didn't count that lane if the sampled point was even with a driveway, crosswalk, outside the marked legal parking area, etc. Human error may apply.

Over my 100 sampled points, I found an average of 0.69 non-metered parking spots along a street at a sampled point.[[^3]] The total length of streets in Cambridge is approximately 250 kilometers = 821,000 feet[[^4]], so we must have around 567,000 linear feet of parking lanes. At 18 feet per spot (marked spots are often more like 20, but go out and measure some cars and you'll often find them closer than 18 feet), that's 31,474 spots.

Fun fact: I also counted bike lanes at my sampled points using analogous logic and found an average of 0.34 bike lanes, meaning if you go to a random spot on the streets of Cambridge you're twice as likely to find nearly free parking as you are a bike lane. Perhaps we should improve upon this.

----

[^1]: Sometimes parking is a fig leaf for other concerns not expressable in polite company.
[^2]: Arguably, actually $0 per year. A visitor parking permit costs $25/year; a resident parking permit also costs $25/year but comes with a visitor parking permit. You can do the subtraction...
[^3]: The standard deviation of this mean is pretty large, around .07, so my estimates could easily be +/- 15%. I wish this were order of magnitude smaller, but that root-n scaling on standard deviations of a population mean really gets ya and realistically I was pretty bored of hand labeling points. [Here's the data](https://docs.google.com/spreadsheets/d/1RfaNiy9ffQXxVF3iaUqUZ_rKnj8DcR9nGHgFtD1n24c/edit?usp=sharing).
[^4]: Anyone got a good efficient bike route that hits them all? Let's ride.