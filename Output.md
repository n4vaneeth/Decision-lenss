#This file contains a sample output done with the test prompt 
```text
(.venv) PS G:\Decision-Lens\src> python main.py  

██████╗ ███████╗ ██████╗██╗███████╗██╗ ██████╗ ███╗   ██╗    ██╗     ███████╗███╗   ██╗███████╗
██╔══██╗██╔════╝██╔════╝██║██╔════╝██║██╔═══██╗████╗  ██║    ██║     ██╔════╝████╗  ██║██╔════╝
██║  ██║█████╗  ██║     ██║███████╗██║██║   ██║██╔██╗ ██║    ██║     █████╗  ██╔██╗ ██║███████╗
██║  ██║██╔══╝  ██║     ██║╚════██║██║██║   ██║██║╚██╗██║    ██║     ██╔══╝  ██║╚██╗██║╚════██║
██████╔╝███████╗╚██████╗██║███████║██║╚██████╔╝██║ ╚████║    ███████╗███████╗██║ ╚████║███████║
╚═════╝ ╚══════╝ ╚═════╝╚═╝╚══════╝╚═╝ ╚═════╝ ╚═╝  ╚═══╝    ╚══════╝╚══════╝╚═╝  ╚═══╝╚══════╝

Decision Companion System
Describe your decision problem:
Type 'done' on a new line when finished.

> I need help deciding what vehicle to buy, and honestly my thoughts are all over the place.
> 
> I live in a city with terrible roads and traffic, but I also do highway trips maybe once or twice a month.
> Fuel prices are killing me, maintenance costs matter, and I don’t want something that feels unsafe.
> At the same time, I don’t want a boring car — driving feel and comfort are important.
>
> Here are the options I’m considering:
>
> 1) Used Toyota Fortuner (2019 diesel)
> This one feels very solid and safe. Comfort is excellent and highway stability is amazing.
> Fuel efficiency isn’t great though — I think it gives around 10 kmpl in the city.
> Maintenance is on the higher side. Price is about 32 lakhs.
> Power is strong, driving feel is very confidence-inspiring.
>
> 2) New Hyundai Creta (diesel)
> Much cheaper than the Fortuner, around 19.5 lakh.
> Fuel efficiency is pretty good, maybe around 17 kmpl.
> Comfort is good, not amazing. Feels safe enough.
> Maintenance should be reasonable.
> Driving feel is decent, not sporty.
>
> 3) Maruti Suzuki Brezza (petrol)
> This is the cheapest option, about 12 lakh.
> Mileage is good, around 18 kmpl.
> Maintenance is very low.
> Comfort is okay, safety is average at best.
> Driving feel is boring but predictable.
>
> 4) Used Skoda Octavia (petrol)
> This one is expensive to maintain but extremely fun to drive.
> Costs around 22 lakh.
> Mileage is bad, maybe 11 kmpl.
> Comfort is very good, safety feels excellent.
> Ground clearance worries me a bit for bad roads.
>
> What I care about (roughly, not in exact numbers):
> - I don’t want very high maintenance
> - Fuel efficiency matters, but not more than safety
> - Comfort and driving feel matter a lot on highways
> - I’m okay paying more if the experience is worth it
> - I don’t want something that constantly scrapes on bad roads
>
> Some things I’m not sure about:
> - Exact maintenance cost numbers
> - Long-term reliability of the used cars
> - How much ground clearance actually matters day to day
>
> Help me think this through.
> done

Identified options:
1. Used Toyota Fortuner (2019 diesel)
2. New Hyundai Creta (diesel)
3. Maruti Suzuki Brezza (petrol)
4. Used Skoda Octavia (petrol)

[a] Add option  [r] Remove option  [c] Continue: c

Identified criteria:
1. price (cost)
2. fuel efficiency (benefit)
3. maintenance cost (cost)
4. comfort (benefit)
5. driving feel (benefit)
6. safety (benefit)
7. ground clearance (cost)

[a] Add criterion  [r] Remove criterion  [c] Continue: c

Confirm criteria types:
1. price → cost
2. fuel efficiency → benefit
3. maintenance cost → cost
4. comfort → benefit
5. driving feel → benefit
6. safety → benefit
7. ground clearance → cost

Type numbers to toggle, 'done' when satisfied.

Enter criterion number to toggle (or 'done'): 7

✔ ground clearance set to benefit

Confirm criteria types:
1. price → cost
2. fuel efficiency → benefit
3. maintenance cost → cost
4. comfort → benefit
5. driving feel → benefit
6. safety → benefit
7. ground clearance → benefit

Type numbers to toggle, 'done' when satisfied.

Enter criterion number to toggle (or 'done'): done

Criterion: maintenance cost
Proposed order (worst → best):
1. very high
2. high
3. reasonable
4. low
5. very low
Is this order correct for you? (y/n): y

Criterion: comfort
Proposed order (worst → best):
1. poor
2. okay
3. good
4. very good
5. excellent
Is this order correct for you? (y/n): y

Criterion: driving feel
Proposed order (worst → best):
1. boring
2. predictable
3. decent
4. good
5. fun
Is this order correct for you? (y/n): y

Criterion: safety
Proposed order (worst → best):
1. poor
2. average
3. safe
4. very safe
5. excellent
Is this order correct for you? (y/n): y

Criterion: ground clearance
Proposed order (worst → best):
1. very low
2. low
3. average
4. high
5. very high
Is this order correct for you? (y/n): n
Enter corrected scale (comma-separated, lowest → highest): low,medium,high

⚠ You defined a scale for 'ground clearance', but no options have values for it.
Would you like to:
[1] Enter values now
[2] Ignore this criterion
Choice (default = 2): 1

Enter values for 'ground clearance':
  Used Toyota Fortuner (2019 diesel) already has a value. Overwrite? (y/n): y
  Used Toyota Fortuner (2019 diesel) (choose from: low, medium, high or type 'skip'): high
  New Hyundai Creta (diesel) already has a value. Overwrite? (y/n): y
  New Hyundai Creta (diesel) (choose from: low, medium, high or type 'skip'): medium
  Maruti Suzuki Brezza (petrol) already has a value. Overwrite? (y/n): y
  Maruti Suzuki Brezza (petrol) (choose from: low, medium, high or type 'skip'): medium
  Used Skoda Octavia (petrol) already has a value. Overwrite? (y/n): y
  Used Skoda Octavia (petrol) (choose from: low, medium, high or type 'skip'): low

Criteria summary (for ranking):
1. price (cost, unit: lakhs)
2. fuel efficiency (benefit, unit: kmpl)
3. maintenance cost (cost) → scale: very high < high < reasonable < low < very low
4. comfort (benefit) → scale: poor < okay < good < very good < excellent
5. driving feel (benefit) → scale: boring < predictable < decent < good < fun
6. safety (benefit) → scale: poor < average < safe < very safe < excellent
7. ground clearance (benefit) → scale: low < medium < high

Rank criteria by importance (1 = highest priority).
Each rank must be UNIQUE (no ties).

price: 3
fuel efficiency: 4
maintenance cost: 5
comfort: 2
driving feel: 1
safety: 6
ground clearance: 7

Ranked Results:
Used Skoda Octavia (petrol): 0.679
Used Toyota Fortuner (2019 diesel): 0.592
New Hyundai Creta (diesel): 0.546
Maruti Suzuki Brezza (petrol): 0.339

--- Explanation ---
The top option is the Used Skoda Octavia (petrol), which ranked highest mainly because it scored well in driving feel, comfort, and maintenance cost.

The Used Toyota Fortuner (2019 diesel) came in second. It also performed well in comfort and driving feel, but it was held back by a higher price and lower fuel efficiency.

The New Hyundai Creta (diesel) ranked third. It had good fuel efficiency and driving feel, but it fell short in safety and comfort compared to the top two options.

Finally, the Maruti Suzuki Brezza (petrol) was ranked fourth. While it excelled in price and fuel efficiency, it struggled with maintenance costs, comfort, and driving feel, which are important factors.

No information was missing in this decision, so all relevant criteria were considered. Your priorities of driving feel, comfort, and price were taken into account, which is why the Skoda Octavia stood out as the best choice for you.
-------------------


What would you like to do next?
[1] Change criteria importance
[2] Add / remove / edit options
[3] Add / remove / edit criteria
[0] Exit
Choice: 0

Decision finalized. Exiting.
(.venv) PS G:\Decision-Lens\src> 
```
