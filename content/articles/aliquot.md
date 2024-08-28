+++
title = "Visualizing Aliquot sequences"
date = 2024-08-28
+++

After watching [this Numberphile video](https://www.youtube.com/watch?v=OtYKDzXwDEE) I got interested in
Aliquot sequences and how I could visualize them differently than in the video.

Aliquot sequences are created by summing the proper divisors of the previous term, ending at 1 since it has no proper divisors.
Not all sequences end at 1 however, these numbers are categorized as follows:

- Perfect numbers: numbers where the sum of it's proper divisors is itself.
- Amicable numbers: numbers that keep going back and forth, e.g. 220 and 284.
- Sociable numbers: sequences that go in a cycle longer that 2 numbers.
- Aspiring numbers: these numbers are not perfect themselves but end at a perfect number instead of 1.

Lastly, there are also divergent numbers, which are numbers of which the full Aliquot sequence has not yet been found. The Lehmer five are the most well known as they are the first 5 numbers for which this is the case. The Lehmer five are: 276, 552, 564, 660, 966.

I want to visualize all the Aliquot sequences up to 1000.
Computing the sequences for these divergent numbers is being done by supercomputers as we speak.
Running a python script on my 10 year old computer is not going to beat that.
So to limit how far we go, we stop computing the sequences after doing 10 steps from the original number.
We create an edge for the last number computed to a ??? sink node, as can be seen in the graph.

The network containing the numbers that connect to 1 are in blue, the unknown sequences are colored red and the other networks are green.

<div class="center">
    <iframe
        src="/aliquot.html"
        width="100%"
        height="800"
        frameBorder="0"
    ></iframe>
</div>
<br>

The graph originally took a long time to load because of the physics being applied, but without physics the graph does not look nice.
So after finding [this issue](https://github.com/WestHealth/pyvis/issues/88), I decided to store the node positions after the physics were applied by doing the following:

1. Open web console
2. Disable physics: `network.setOptions( { physics: false } );`
3. Extract node positions:
```javascript
var result = {};
for (const [key, value] of Object.entries(network.body.nodes)) {
    if (!key.includes("edge")) {
        result[key] = [value.x, value.y];
    }
}
console.log(JSON.stringify(result));
```

This makes the graph load instantly and still maintains the positions!
The python code used to generate the graph can be found [in this gist](https://gist.github.com/WJehee/4d708b111190554fa88e55d517050b20)
(you need to have [networkx](https://networkx.org/) and [pyvis](https://pyvis.readthedocs.io/en/latest/index.html) installed.

