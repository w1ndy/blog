---
title: Generating Colors for Tree Structured Data
date: 2015-12-28 02:27:40
tags:
  - visualization
  - colors
categories:
  - visualization
---

The algorithm used here is based on M. Tennekes, et al.'s work: [*Tree Colors: Color Schemes for Tree-Structured Data*](https://www.computer.org/csdl/trans/tg/preprint/06875961.pdf) specifically designed for generating colors for tree structured data (like tree maps) in [HCL](https://en.wikipedia.org/wiki/HCL_color_space) color space. A HCL color is made of three components: hue, chroma and luminance. Hue defines the color we commonly refer to in degree (from 0 to 360), chroma interpolates color from gray to full saturation, and luminance decides how bright the color looks.

Generating procedure can be divided into 2 parts:
<!-- more -->

1. Hue
    A recursive method is defined as following:

    > AssignHue(v, r, f, perm, rev)

    where ``v`` is a node in the tree; ``r`` is the hue range that node ``v`` occupies; ``f`` is the fraction of the hue range used (separating the hue range from its siblings); ``perm`` decides whether to permutate the colors generated; ``rev`` decides whether to reverse even-numbered colors to fully obfuscate the sequence.

    * Color Assignment
        Node ``v``'s hue is the mid point of hue range ``r``, i.e.:

        ```js
        v.color.h = (r.left + r.right) / 2
        ```

        Then ``AssignHue`` divided the hue range for each of it's child node, and shrink by the fraction defined (suggested 0.5 by the paper).

    * Permutation
        Calculate a permutation for the hue range sequence such that every hue range is at least twice of its length away from its nearest siblings (N >= 5) to prevent a perceptual order of the siblings. For example, a parent hue range is divided equally among 5 children, producing a hue range sequence [1, 2, 3, 4, 5]. Permutate the sequence we get [1, 3, 5, 2, 4]. For 3 or 4 children, we would achieve [1, 3, 2, 4] and [1, 3, 2] by the same method. Simply we can take out odd-numbered elements and concatenate with even-numbered elements.

    * Reversal
        To avoid color collision between two categories that both contain the even number of children (in this case, the child with the highest hue in the first category is close to the one with the least in the second category), the algorithm perform the reversal on even-numbered hue ranges, that is, [1, 3, 2, 4] becomes [1, 4, 2, 3].

2. Chroma and Luminance
    The algorithm uses linear interpolation for chroma and luminance:

    > C = (d - 1) * bC + C1
    > L = (d - 1) * bL + L1

    where d is the depth of the node, bC, bL are the sloppery for the components, and C1, L1 are the initial value (for tree nodes of depth 1). The values suggested in the paper are bC = 5, C1 = 60, bL = -10, L1 = 70.

Implementation in JavaScript:

```javascript
var svg = d3.select("#graph").append("svg")
        .attr("width", 600)
        .attr("height", 400),
    L1 = 70, bL = -10,
    C1 = 60, bC = 5;

function createTree(maxDepth, nChildren) {
    function generator(depth) {
        var node = {
                children: [],
                color: null
            };
        if (depth == maxDepth)
            return node;
        for (var i = 0; i < nChildren; i++) {
            node.children.push(generator(depth + 1));
        }
        return node;
    }
    return generator(0);
}

function calcColor(v, h, f, perm, rev, depth) {
    if (depth == 0)
        v.color = d3.hcl(0, 0, L1 - bL);
    if (!v.children)
        return ;

    var sub = [],
        len = (h[1] - h[0]) / v.children.length,
        margin = 0.5 * (1 - f) * len;

    for (var i = h[0]; i < h[1]; i += len)
        sub.push([i + margin, i + len - margin]);

    // permute colors
    if (perm) {
        var old = sub;
        sub = [];
        for (var i = 0; i < old.length; i += 2)
            sub.push(old[i]);
        for (var i = 1; i < old.length; i += 2)
            sub.push(old[i]);
    }

    // reverse color sequence on even positions
    if (rev) {
        var last = sub.length - 1, first = 1;
        if (last % 2 == 0) last -= 1;
        while(first < last) {
            var tmp = sub[first];
            sub[first] = sub[last];
            sub[last] = tmp;
            first += 2;
            last -= 2;
        }
    }

    for (var i = 0; i < v.children.length; i++) {
        v.children[i].color = d3.hcl(
            (sub[i][0] + sub[i][1]) / 2,
            C1 + depth * bC,
            L1 + depth * bL);
        calcColor(v.children[i], sub[i], f, perm, rev, depth + 1);
    }
}

function graphizeTree(rootNode) {
    var graph = {
            nodes: [],
            links: []
        };
    function traverse(node, parent) {
        var id = graph.nodes.length;
        graph.nodes.push({
            id: id,
            color: node.color
        });
        if (parent !== undefined) {
            graph.links.push({
                source: id,
                target: parent
            });
        }
        for (var i = 0; i < node.children.length; i++) {
            traverse(node.children[i], id);
        }
    }
    traverse(rootNode);
    return graph;
}

function main() {
    var tree, graph;

    tree = createTree(3, 3);
    calcColor(tree, [0, 360], 0.5, true, true, 0);
    graph = graphizeTree(tree);

    var layout = d3.layout.force()
            .charge(-120)
            .linkDistance(30)
            .size([600, 400])
            .nodes(graph.nodes)
            .links(graph.links),
        link = svg.selectAll(".link")
                .data(graph.links)
            .enter().append("line")
                .attr("class", "link")
                .attr("stroke", "#999")
                .attr("stroke-width", 2),
        node = svg.selectAll(".node")
                .data(graph.nodes)
            .enter().append("circle")
                .attr("class", "node")
                .attr("stroke", "#fff")
                .attr("stroke-width", 1.5)
                .attr("fill", function (d) { return d.color; })
                .attr("r", 5)
                .call(layout.drag);
    layout.start();
    layout.on("tick", function () {
        link.attr("x1", function(d) { return d.source.x; })
            .attr("y1", function(d) { return d.source.y; })
            .attr("x2", function(d) { return d.target.x; })
            .attr("y2", function(d) { return d.target.y; });
        node.attr("cx", function(d) { return d.x; })
            .attr("cy", function(d) { return d.y; });
    });
}

main();
```