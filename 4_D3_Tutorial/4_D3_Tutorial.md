# IV. Multiple Views and Advanced Interactivity

#### Learning objectives

* Linked interactions
* Brushing and linking
* Interactions across multiple charts (event handling via controller)
* D3 shape generators
* Color and size legends

#### Contents

1. [Enter, Update, Exit](#enter-update-exit)
2. [Making a bar chart](#more-d3-basics)
3. [Reusable D3 components](#reusable-d3-components)
4. [Making line and area charts](#making-line-and-area-charts)
5. [Tooltips](#tooltips)


In the last few weeks, you have learned the fundamentals of D3 and gained some implementation expertise during exercises and the first homework. You should be comfortable with the major concepts and be able to implement common charts as well as interactive and more advanced visualizations with D3.

## Linked interactions

Visualizations are often not just single charts and especially for more complex analysis tasks it is important to facet data across multiple coordinated views.

### Basic linkage

We will first walk through a very basic mechanism to link two charts. This is good enough for some cases but more complex visualizations necessitate an event handling controller to ensure good programming practice.

We want to visualize hiking trails near Vancouver, as illustrated in the figure below. The first chart shows a scatter plot and the difficulty level (*easy*, *intermediate*, *difficult*) is color-coded. The number of hikes in these three categories is not immediately obvious without counting each dot. For this purpose, we add a bar chart that can simultaneously serve as a filter. When users click on one of the bars, the data in the scatter plot gets filtered automatically.

todo: figure

1. **Setup**

	We create three JS files, `main.js`, `scatterplot.js`, and `barchart.js`, in addition to the HTML and CSS files.
	
	* In `main.js`, we load the data and initialize the two vis classes:

		```js
		let data, scatterplot, barchart;
		```
		
		```js
		d3.csv('data/vancouver_trails.csv')
		  .then(_data => {
		    data = _data;
		    
		    // ... data preprocessing etc. ...
		    
		    scatterplot = new Scatterplot(config, data);
		    scatterplot.updateVis();
		
		    barchart = new Barchart(config, data);
		    barchart.updateVis();
		  });
		```
		
	* We implement two classes `Scatterplot` and `Barchart` as learned earlier.   For the scatter plot, it is important that we use D3's enter-update-exit pattern because we don't want to remove and redraw the entire chart whenever the data changes.

2. **Filtering mechanism** in `main.js`

	We add a global array to store active filter options.
	
	```js
	let difficultyFilter = [];
	```
	
	Filter the data in the scatter plot accordingly (all data is shown when no filters are selected).
	
	```js
	function filterData() {
	  if (difficultyFilter.length == 0) {
	    scatterplot.data = data;
	  } else {
	    scatterplot.data = data.filter(d => difficultyFilter.includes(d.difficulty));
	  }
	  scatterplot.updateVis();
	}
	```

3. **Add event listener** in `barchart.js`

	Whenever users click on a *bar* we want to update the selection and call `filterData()` to trigger a change in the scatter plot.
	
	```js
	const bars = vis.chart.selectAll('.bar')
        .data(vis.aggregatedData, vis.xValue)
      .join('rect')
        .attr('class', 'bar')
        .attr('x', d => vis.xScale(vis.xValue(d)))
        // ... other attributes ... 
        .on('click', function(event, d) {
        	// Check if filter is already active
          const isActive = difficultyFilter.includes(d.key);
          if (isActive) { 
            // Remove filter
            difficultyFilter = difficultyFilter.filter(f => f !== d.key);
          } else { 
            // Add filter
            difficultyFilter.push(d.key);
          }
          // Call global function to update scatter plot
          filterData();
          
          // Add class to style active filters with CSS
          d3.select(this).classed('active', !isActive);
        });
	```

4. **Style bar chart filters** in `style.css`

	```css
	.bar:hover {
	  stroke: #777;
	}
	.bar.active {
	  stroke: #333;
	}
	```

You can see the final source code of this example here: [todo](todo)

## D3 Layouts

Visualizations typically consist of discrete graphical marks, such as circles, rectangles, symbols, arcs, lines and areas. While the rectangles of a bar chart or the points in a scatter plot may be easy enough to generate directly using SVG, other shapes are more complex.

D3 provides functions — so-called *shape generators* — to help us with the creation of more complex shapes. In the 2. Tutorial we have already introduced two of those shape/line generators (`d3.line()` and `d3.area()`) when we showed you how to create line and area charts.

The D3 shape generators have no direct visual output but instead take the data you provide and transform it, thereby generating new data that is more convenient to draw.

#### Symbols

Symbols are commonly used in scatter plots. We can create them using the `d3.symbol()` function.

![D3 Symbols](d3-symbols.png?raw=true "D3 Symbols")

For example, we can create an ordinal scale to show different symbols for the three categories *"Easy"*, *"Intermediate"*, and *"Difficult"*.


#### Stacks

D3 offers a number of different layouts, each with distinct characteristics. Layouts are invoked using *d3.layout*:

### Hierarchy

See documentation for arcs etc.

